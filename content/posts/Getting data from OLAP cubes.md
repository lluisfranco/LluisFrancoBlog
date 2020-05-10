---
title: "Getting data from OLAP cubes"
subtitle: "Using ADOMD.NET Dapper style"
date: 2017-12-12T11:51:50+02:00
featuredImage: /images/posts/olap.png
fontawesome: "true"
categories: 
- Development
- How To
tags:
- csharp
- net framework
- OLAP
- ADOMD.NET
- generics
# draft: true
---
## Overview

Recently I had to read some data from different OLAP cubes and show this information in our application. This could be accomplished using [ADOMD.NET](https://docs.microsoft.com/en-us/analysis-services/adomd/developing-with-adomd-net?view=asallproducts-allversions) client library, which is the multidimensional equivalent to the ADO.NET libraries.

This library provides a set of classes (Connections, Commands and Readers) that allow developers to connect, query and get data from these multidimensional sources in the same way the classical ADO.NET library does.

Maybe the easiest way to read data is using the class [AdomdDataReader](https://docs.microsoft.com/en-us/dotnet/api/microsoft.analysisservices.adomdclient.adomddatareader?view=analysisservices-dotnet). This object retrieves a read-only, forward-only, flattened stream of data from an analytical data source. Also implements the IDataReader interface (like the good-old DataReader).

In non dynamic scenarios, we developers, tend to use ORMs like NHibertate, EF or Dapper (my favorite), to translate data from our models to classes. But these ORMs doesn’t support multidimensional sources, because after all, they are only supersets or extensions of the ADO.NET model and are based on this existing object model.

So, our goal here will be replicate in a multidimensional environment the way Dapper allows you to connect to a server, read some data and (the most important part) return these data as a typed List<T> of objects.

## Testing the query

Let’s see how it works before dig into the details:

{{< highlight csharp "linenos=table,hl_lines=4-12" >}}
using (var conn = new AdomdConnection(connectionString))
{
    conn.Open();
    var commandText = @"SELECT
                        {
                        [Measures].[Gains Period],
                        [Measures].[Gains YTD],
                        [Measures].[Amount]
                        } ON COLUMNS,
                        [Valuation Dates Accumulated].[Hierarchy].[Year].Members ON ROWS
                        FROM Accumulated
                        Where ({ @PortfolioId })";
    var data = conn.Query<MyResultDTO>(commandText,
        new AdomdParameterMultipleValues<int>()
        {
            ParameterName = "@PortfolioId",
            MemberName = "[Portfolios].[Port Code]",
            Values = new List<int>() { 282, 185 }
        }
    );
    conn.Close();
}
{{< / highlight >}}

First of all we have a SQL script (highligted lines above) that returns some sample data of 3 different measures year by year (gains in period, gains year-to-date and the amount of money).

{{< figure src="/images/posts/mddata.png" title="Sample data" >}}

Our goal here is translate these data into a generic List of our own DTO:

{{< highlight csharp "linenos=table" >}}
internal class MyResultDTO
{
    [OLAPMemberNameAttribute("[Valuation Dates Accumulated].[Hierarchy].[Year].[MEMBER_CAPTION]")]
    public string Year { get; set; }
    [OLAPMemberNameAttribute("[Measures].[Gains Period]")]
    public decimal? GainsPeriod { get; set; }

    [OLAPMemberNameAttribute("[Measures].[Gains YTD]")]
    public decimal? GainsYTD { get; set; }

    [OLAPMemberNameAttribute("[Measures].[Amount]")]
    public decimal? Amount { get; set; }
}
{{< / highlight >}}

> EDIT (2018-12-11) : Added custom attribute to the post, used to decorate our DTO.

{{< highlight csharp "linenos=table" >}}
public class OLAPMemberNameAttribute : System.Attribute
{
    string _Description = string.Empty;
    public OLAPMemberNameAttribute(string description)
    {
        _Description = description;
    }
    public string Description { get { return _Description; } }
}
{{< / highlight >}}

Notice that in C# properties can not contain spaces so we previously have to ‘map’ the DataReader column names into valid property names. The trick here is create an attribute for decorating each property with the equivalent column name in the reader.

The magic here is inside the generic Query<T> Method. This method is just an extension method of the AdomdConnection object (Dapper style) and it’s mission will be:

1. Replace the parameters in the command string “...Where ({ @PortfolioId })”
2. Execute the command against a valid connection
3. Transform the returned IDataReader into a List<MyResultDTO>

{{< highlight csharp "linenos=table" >}}
public static List<T> Query<T>(
    this AdomdConnection conn, string commandText, params object[] parameters)
{
    var commandTextParameters = ReplaceCommandTextParameters(commandText, parameters);
    var cmd = new AdomdCommand(commandTextParameters, conn);
    var dr = cmd.ExecuteReader();
    return dr.ToList<T>();
}
{{< / highlight >}}

Now let's focus on the third point and create another extension method of the IDataReader interface. This method will transform the returned result set into a generics List<T>

OMG dude! Code on interfaces? Yes... I’ll burn in hell, I know it :smile:

{{< highlight csharp "linenos=table" >}}
public static List<T> ToList<T>(this IDataReader dr)
{
    var list = new List<T>();
    var obj = default(T);
    while (dr.Read())
    {
        obj = Activator.CreateInstance<T>();
        foreach (var prop in obj.GetType().GetProperties())
        {
            var columnDescription = getPropertyDescription(obj, prop.Name);
            if (existsDataReaderColumn(dr, columnDescription)) copyColumnValueToProperty(dr, obj, prop);
        }
        list.Add(obj);
    }
    return list;
}

static void copyColumnValueToProperty<T>(IDataReader dr, T obj, PropertyInfo prop)
{
    try
    {
        var columnDescription = getPropertyDescription(obj, prop.Name);
        var columnOrdinal = dr.GetOrdinal(columnDescription);
        var value = dr[columnOrdinal];
        var canBeNull = !prop.PropertyType.IsValueType || (Nullable.GetUnderlyingType(prop.PropertyType) != null);
        var castToType = Nullable.GetUnderlyingType(prop.PropertyType) ?? prop.PropertyType;
        if (canBeNull && value == null)
            prop.SetValue(obj, null, null);
        else
            prop.SetValue(obj, Convert.ChangeType(value, castToType, CultureInfo.InvariantCulture), null);
    }
    catch { }
}

static bool existsDataReaderColumn(IDataReader dr, string propertyName)
{
    try
    {
        var obj = dr[propertyName];
        return true;
    }
    catch { return false; }
}

private static string getPropertyDescription(object value, string propname)
{
    var propinfo = value.GetType().GetProperty(propname);
    var attributes =
        (OLAPMemberNameAttribute[])propinfo.GetCustomAttributes(
        typeof(OLAPMemberNameAttribute), false);
    if (attributes != null && attributes.Length > 0)
        return attributes[0].Description;
    else
        return value.ToString();
}
{{< / highlight >}}

As you can see the method iterates over the reader’s data and uses reflection to create an object per each row.
Then map each column’s value to a property and add the new object to the returned collection (*). Easy!

{{< figure src="/images/posts/adomd_result.png" title="Returned data as List<T>" >}}

(*) image courtesy of oz-code, the best debugger addin for visual studio #imho ;)

Hope you enjoy it!
