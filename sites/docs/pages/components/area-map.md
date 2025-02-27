---
title: Area Map
sidebar_position: 1
---

<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='https://evd-geojson.b-cdn.net/ca_california_zip_codes_geo_1.min.json'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
/>

```svelte
<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='path/to/your/geoJson'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
/>
```


```sql la_zip_sales
select *, 'https://www.google.com/search?q=' || zip_code as link_col from la_zip_sales
where zip_code <> 90704
```


## Examples

### Custom Basemap
You can add a different basemap by passing in a basemap URL. You can find examples here: https://leaflet-extras.github.io/leaflet-providers/preview/

<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='https://evd-geojson.b-cdn.net/ca_california_zip_codes_geo_1.min.json'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
    borderColor=#303030
    basemap={`https://tile.openstreetmap.org/{z}/{x}/{y}.png`}
/>

**Note:** you need to wrap the url in curly braces and backticks to avoid the curly braces in the URL being read as variables on your page

```svelte
<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='path/to/your/geoJson'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
    basemap={`https://tile.openstreetmap.org/{z}/{x}/{y}.png`}
/>
```

### Using an Online GeoJSON

```sql orders_by_state
select state, count(*) as orders
from orders
where state != 'Alaska' and state != 'Hawaii'
group by state
```

<AreaMap 
    data={orders_by_state} 
    areaCol=state
    geoJsonUrl=https://d2ad6b4ur7yvpq.cloudfront.net/naturalearth-3.3.0/ne_110m_admin_1_states_provinces.geojson
    geoId=name
    value=orders
/>

```svelte
<AreaMap 
    data={orders_by_state} 
    areaCol=state
    geoJsonUrl=https://d2ad6b4ur7yvpq.cloudfront.net/naturalearth-3.3.0/ne_110m_admin_1_states_provinces.geojson
    geoId=name
    value=orders
/>
```

### Custom Tooltip

#### `tooltipType=hover`
<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='https://evd-geojson.b-cdn.net/ca_california_zip_codes_geo_1.min.json'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
    tooltip={[
        {id: 'zip_code', fmt: 'id', showColumnName: false, valueClass: 'text-xl font-semibold'},
        {id: 'sales', fmt: 'eur', fieldClass: 'text-[grey]', valueClass: 'text-[green]'}
    ]}
/>

```svelte
<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='path/to/your/geoJson'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
    tooltip={[
        {id: 'zip_code', fmt: 'id', showColumnName: false, valueClass: 'text-xl font-semibold'},
        {id: 'sales', fmt: 'eur', fieldClass: 'text-[grey]', valueClass: 'text-[green]'}
    ]}
/>

```

#### With clickable link and `tooltipType=click`
<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='https://evd-geojson.b-cdn.net/ca_california_zip_codes_geo_1.min.json'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
    tooltipType=click
    tooltip={[
        {id: 'zip_code', fmt: 'id', showColumnName: false, valueClass: 'text-xl font-semibold'},
        {id: 'sales', fmt: 'eur', fieldClass: 'text-[grey]', valueClass: 'text-[green]'},
        {id: 'link_col', showColumnName: false, contentType: 'link', linkLabel: 'Click here', valueClass: 'font-bold mt-1'}
    ]}
/>


```svelte
<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='path/to/your/geoJson'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
    tooltipType=click
    tooltip={[
        {id: 'zip_code', fmt: 'id', showColumnName: false, valueClass: 'text-xl font-semibold'},
        {id: 'sales', fmt: 'eur', fieldClass: 'text-[grey]', valueClass: 'text-[green]'},
        {id: 'link_col', showColumnName: false, contentType: 'link', linkLabel: 'Click here', valueClass: 'font-bold mt-1'}
    ]}
/>
```

### Custom Styling

<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='https://evd-geojson.b-cdn.net/ca_california_zip_codes_geo_1.min.json'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
    color=#fff5d9
    borderColor=#737373
    borderWidth=0.5
/>

```svelte
<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='path/to/your/geoJson'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
    color=#fff5d9
    borderColor=#737373
    borderWidth=0.5
/>
```

### Custom Color Palette

<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='https://evd-geojson.b-cdn.net/ca_california_zip_codes_geo_1.min.json'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
    colorPalette={['yellow','orange','red','darkred']}
/>

```svelte
<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='path/to/your/geoJson'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
    colorPalette={['yellow','orange','red','darkred']}
/>
```


### Link Drilldown
Pass in a `link` column to enable navigation on click of the point. These can be absolute or relative URLs

<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='https://evd-geojson.b-cdn.net/ca_california_zip_codes_geo_1.min.json'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
    link=link_col
/>

```svelte
<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='path/to/your/geoJson'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
    link=link_col
/>
```


### Use Map as Input
Use the `name` prop to set an input name for the map - when a point is clicked, it will set the input value to that row of data

```svelte
<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='path/to/your/geoJson'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
    name=my_area_map
/>
```

<AreaMap 
    data={la_zip_sales} 
    areaCol=zip_code
    geoJsonUrl='https://evd-geojson.b-cdn.net/ca_california_zip_codes_geo_1.min.json'
    geoId=ZCTA5CE10
    value=sales
    valueFmt=usd
    height=250
    name=my_area_map
/>


*Click an area on the map to see the input value get updated:*

#### Selected value for `{inputs.my_area_map}`: 

<pre class="text-sm">{JSON.stringify(inputs.my_area_map, null, 2)}</pre>

#### Selected value for `{inputs.my_area_map.zip_code}`: 
  
{inputs.my_area_map.zip_code}


```filtered_areas
select * from ${la_zip_sales}
where zip_code = ${inputs.my_area_map.zip_code} OR ${inputs.my_area_map.zip_code} = true
```

#### Filtered Data
<DataTable data={filtered_areas}>  	
    <Column id=id/> 	
    <Column id=zip_code fmt=id/> 	
    <Column id=sales fmt=usd/> 	
</DataTable>

## Required GeoJSON Data Structure
The GeoJSON data you pass to the map must be a feature collection. [See here for an example](https://gist.github.com/sgillies/1233327#file-geojson-spec-1-0-L50)

## Map Resources

```sql all_geojson_urls
select * exclude(properties)
from geojson_urls
order by scale, category, file
```

```sql useful_geojson_urls
select * 
from ${all_geojson_urls}
where category in ('political_countries', 'political_states')
or file ilike 'populated_places%'
order by scale desc, category, file
```

Below are a selection of publically available GeoJSON files that may be useful for mapping. These are from the [Natural Earth Data](https://www.naturalearthdata.com/) project, and hosted by [GeoJSON.xyz](https://geojson.xyz/).

### Country, State, and City Locations

<DataTable data={useful_geojson_urls} rows=100>
    <Column id=file/>
    <Column id=category/>
    <Column id=scale/>
    <Column id=summary/>
    <Column id=size fmt='0.0,," MB"'/>
    <Column id=url contentType=link title=URL/>
</DataTable>

<Details title="All GeoJSON Files">

<DataTable data={all_geojson_urls} rows=all compact>
    <Column id=file/>
    <Column id=category/>
    <Column id=scale/>
    <Column id=summary/>
    <Column id=size fmt='0.0,," MB"'/>
    <Column id=url contentType=link title=URL/>
</DataTable>

</Details>

## Options

### Areas
<PropListing
name="data"
required    
options="query name"
>
Query result, referenced using curly braces
</PropListing>

<PropListing
name="geoJsonUrl"
required
options="URL"
>

Path to source geoJSON data from - can be a URL (see [Map Resources](#map-resources)) or a file in your project. 

If the file is in your project, store it in a `static` folder in the root of your project, and reference it as `geoJsonUrl="/your_file.geojson"`

</PropListing>

<PropListing
name="areaCol"
required
options="column name"
>
Column in the data that specifies the area each row belongs to.
</PropListing>

<PropListing
name="geoId"
required
options="geoJSON property name"
>
Property in the GeoJSON that uniquely identifies each feature.
</PropListing>

<PropListing
name="value"
options="column name"
>
Column that determines the value displayed for each area (used for color scale)
</PropListing>

<PropListing
name="valueFmt"
options="format string"
>
Format string for displaying the value.
</PropListing>

### Color Scale

<PropListing
name="colorPalette"
options="array of colors"
>
Array of colors used for theming the areas based on data <code></code>
</PropListing>

<PropListing
name="min"
options="number"
defaultValue="min of value column"
>
Minimum value to use for the color scale.
</PropListing>

<PropListing
name="max"
options="number"
defaultValue="max of value column"
>
Maximum value to use for the color scale.
</PropListing>

### Interactivity

<PropListing
name="link"
options="URL"
>
URL to navigate to when a area is clicked.
</PropListing>

<PropListing
name="name"
options="string"
>
Input name. Can be referenced on your page with `{inputs.my_input_name}`
</PropListing>

### Styling
<PropListing
name="color"
options="CSS color value"
>
Color for the areas. Use when you want all areas to be the same color.
</PropListing>

<PropListing
name="borderWidth"
options="pixel value"
defaultValue=0.75
>
Width of the border around each area.
</PropListing>

<PropListing
name="borderColor"
options="CSS color value"
defaultVallue="white"
>
Color of the border around each area.
</PropListing>

<PropListing
name="opacity"
options="number between 0 and 1"
defaultValue=0.8
>
Opacity of the areas.
</PropListing>

### Selected State

<PropListing
name="selectedColor"
options="CSS color value"
>
When area is selected: Color for the areas. Use when you want all areas to be the same color.
</PropListing>

<PropListing
name="selectedBorderWidth"
options="pixel value"
defaultValue=0.75
>
When area is selected: Width of the border around each area.
</PropListing>

<PropListing
name="selectedBorderColor"
options="CSS color value"
defaultVallue="white"
>
When area is selected: Color of the border around each area.
</PropListing>

<PropListing
name="selectedOpacity"
options="number between 0 and 1"
defaultValue=0.8
>
When area is selected: Opacity of the areas.
</PropListing>


### Tooltips
<PropListing
name="showTooltip"
options={['true', 'false']}
defaultValue=true
>
Whether to show tooltips
</PropListing>

<PropListing
name="tooltipType"
options={['hover', 'click']}
defaultValue='hover'
>
Determines whether tooltips are activated by hover or click.
</PropListing>

<PropListing
name="tooltipClass"
options="CSS class"
>
CSS class applied to the tooltip content. You can pass Tailwind classes into this prop to custom-style the tooltip
</PropListing>

<PropListing
name="tooltip"
options="array of objects"
>
Configuration for tooltips associated with each area. See below example for format
</PropListing>

<LineBreak/>

#### `tooltip` example:

```javascript
tooltip={[
    {id: 'zip_code', fmt: 'id', showColumnName: false, valueClass: 'text-xl font-semibold'},
    {id: 'sales', fmt: 'eur', fieldClass: 'text-[grey]', valueClass: 'text-[green]'},
    {id: 'zip_code', showColumnName: false, contentType: 'link', linkLabel: 'Click here', valueClass: 'font-bold mt-1'}
]}
```

#### All options available in `tooltip`:
- `id`: column ID
- `title`: custom string to use as title of field
- `fmt`: format to use for value
- `showColumnName`: whether to show the column name. If `false`, only the value will be shown
- `contentType`: currently can only be "link"
- `linkLabel`: text to show for a link when contentType="link"
- `formatColumnTitle`: whether to automatically uppercase the first letter of the title. Only applies when `title` not passed explicitly
- `valueClass`: custom Tailwind classes to style the values
- `fieldClass`: custom Tailwind classes to style the column names



### Base Map

<PropListing
name="basemap"
options="URL"
>
URL template for the basemap tiles.
</PropListing>

<PropListing
name="title"
options="text"
>
Optional title displayed above the map.
</PropListing>

<PropListing
name="startingLat"
options="latitude coordinate"
>
Starting latitude for the map center.
</PropListing>

<PropListing
name="startingLong"
options="longitude coordinate"
>
Starting longitude for the map center.
</PropListing>

<PropListing
name="startingZoom"
options="number (1 to 18)"
>
Initial zoom level of the map.
</PropListing>

<PropListing
name="height"
options="pixel value"
defaultValue="300"
>
Height of the map in pixels.
</PropListing>