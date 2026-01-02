# Properies Review

## Properties Table Structure

```properties_summary
select * from properties.properties
```


| **Column** | **Data Type** | **Description** |
| ------ | --------- | ----------- |
| property_id | number | unique id assigned to each property record |
| property_name | string | name of the property |
| property_type | string | categorical description of properties mode of use | 
| city | string | city the property is located in |
| state | string | two-letter abbreviation of state property is located in |
| total_sqft | number | total sqft of the property |
| year_built | number | 4-digit year the property was built | 


## Properties Data Profiling

### 1. Row Level & Structure Checks

#### Total number of property records
```properties_total_records
select count(*) total from properties.properties
```
- **<Value data={properties_total_records} column=total />** total property records


#### Null / Blanks summary
```properties_null_blank
select
    column_name,
    total_rows,
    null_or_blank_count,
    round(null_or_blank_count / total_rows, 2) null_or_blank_pct
from (
    select
        'property_id' as column_name,
        count(*) as total_rows,
        sum(case when property_id is null then 1 else 0 end) as null_or_blank_count
    from properties.properties
    union all
    select
        'property_name' as column_name,
        count(*) as total_rows,
        sum(case when property_name is null or property_name = '' then 1 else 0 end) as null_or_blank_count
    from properties.properties
    union all
    select
        'property_type' as column_name,
        count(*) as total_rows,
        sum(case when property_type is null or property_type = '' then 1 else 0 end) as null_or_blank_count
    from properties.properties
    union all
    select
        'city' as column_name,
        count(*) as total_rows,
        sum(case when city is null or city = '' then 1 else 0 end) as null_or_blank_count
    from properties.properties
    union all
    select
        'state' as column_name,
        count(*) as total_rows,
        sum(case when state is null or state = '' then 1 else 0 end) as null_or_blank_count
    from properties.properties
    union all
    select
        'total_sqft' as column_name,
        count(*) as total_rows,
        sum(case when total_sqft is null then 1 else 0 end) as null_or_blank_count
    from properties.properties
    union all
    select
        'year_built' as column_name,
        count(*) as total_rows,
        sum(case when year_built is null then 1 else 0 end) as null_or_blank_count
    from properties.properties
)
```
<DataTable data={properties_null_blank} rowNumbers=true rowShading=true>
    <Column id=column_name />
    <Column id=total_rows />
    <Column id=null_or_blank_count contentType=colorscale colorScale=info align=center/>
    <Column id=null_or_blank_pct contentType=colorscale colorScale=info align=center />
</DataTable>

### 2. Primary key uniquness validation

#### Uniqueness of primary key
```properties_unique_pk
select count(*) total_records, count(distinct property_id) unique_property_ids
from properties.properties
```
<Value data={properties_unique_pk} column=total_records /> total property records with <Value data={properties_unique_pk} column=unique_property_ids /> unique property_id values. 

### 3. Foreign key integrity checks

No foreign keys in properties table

## Data Consistency Checks

### 4. Date Field Validation

#### Earliest & Latest date values

*year_built*
```properties_min_max_year_built
select min(year_built) min_year_built, max(year_built) max_year_built
from properties.properties
```
<BigValue data={properties_min_max_year_built} value=min_year_built title="Earliest year_built" fmt="####" />
<BigValue data={properties_min_max_year_built} value=max_year_built title="Latest year_built" fmt="####"/>

#### Check for future start dates
```properties_future_year_built
select count(*) total
from properties.properties
where year_built > year(current_date)
```
<Value data={properties_future_year_built} column=total /> property records with future year_built values.

#### Check for extremely old properties
```properties_oldest_property
select year(current_date) - (select min(year_built) from properties.properties) years_old
```
Oldest property is <Value data={properties_oldest_property} column=years_old /> years old.

#### Property age distribution
```properties_duration
select year(current_date) - year_built age
from properties.properties
```
<Histogram data={properties_duration} x=age/>


### 5. Numeric Field Validation

```properties_numeric_min_max_avg
select
    'total_sqft' column_name,
    min(total_sqft) min, 
    max(total_sqft) max,
    mean(total_sqft) avg,
    sum(case when total_sqft = 0 or total_sqft is null then 1 else 0 end) null_or_zero_count
from properties.properties
union all 
select
    'year_built' column_name,
    min(year_built) min, 
    max(year_built) max,
    mean(year_built) avg,
    sum(case when year_built = 0 or year_built is null then 1 else 0 end) null_or_zero_count
from properties.properties
```
<DataTable data={properties_numeric_min_max_avg} rowNumbers=true rowShading=true>
    <Column id=column_name align=center />
    <Column id=min />
    <Column id=max />
    <Column id=avg />
    <Column id=null_or_zero_count />
</DataTable>

*total_sqft*
```total_sqft_dist
select total_sqft from properties.properties
```
<Histogram data={total_sqft_dist} x=total_sqft />

*year_built*
```year_built_dist
select year_built from properties.properties
```
<Histogram data={year_built_dist} x=year_built />


### 6. Text Field Validation

```properties_text_max_avg
select 
    'property_name' column_name, 
    max(length(property_name)) max_length, 
    mean(length(property_name)) avg_length, 
    sum(case when property_name = '' then 1 else 0 end) empty_string_count, 
    sum(case when property_name is null then 1 else 0 end) null_count
from properties.properties
union all 
select 
    'property_type' column_name, 
    max(length(property_type)) max_length, 
    mean(length(property_type)) avg_length, 
    sum(case when property_type = '' then 1 else 0 end) empty_string_count, 
    sum(case when property_type is null then 1 else 0 end) null_count
from properties.properties
union all
select 
    'city' column_name, 
    max(length(city)) max_length, 
    mean(length(city)) avg_length, 
    sum(case when city = '' then 1 else 0 end) empty_string_count, 
    sum(case when city is null then 1 else 0 end) null_count
from properties.properties
union all
select 
    'state' column_name, 
    max(length(state)) max_length, 
    mean(length(state)) avg_length, 
    sum(case when state = '' then 1 else 0 end) empty_string_count, 
    sum(case when state is null then 1 else 0 end) null_count
from properties.properties
```
<DataTable data={properties_text_max_avg} rowNumbers=true rowShading=true>
    <Column id=column_name align=center />
    <Column id=max_length />
    <Column id=avg_length />
    <Column id=empty_string_count />
    <Column id=null_count />
</DataTable>

```properties_text_common
select
    column_name,
    text_label,
    total,
    rn
from (
    select
        *,
        row_number() over(partition by column_name order by total desc) rn
    from (
    select
        'property_name' column_name,
        property_name text_label,
        count(*) total,
        1 fop
    from properties.properties
    group by property_name
    union all
    select
        'property_type' column_name,
        property_type text_label,
        count(*) total,
        2 fop
    from properties.properties
    group by property_type
    union all
    select
        'city' column_name,
        city text_label,
        count(*) total,
        3 fop
    from properties.properties
    group by city
    union all
    select
        'state' column_name,
        state text_label,
        count(*) total,
        4 fop
    from properties.properties
    group by state
    )
)
where rn <= 5
order by fop, rn
```
<DataTable data={properties_text_common} groupBy=column_name rowNumbers=true rowShading=true>
    <Column id=text_label title="Text" />
    <Column id=total title="Occurrences" />
</DataTable>

*property_type*
```property_type_dist
select property_type, count(*) count
from properties.properties
group by property_type
```
<DataTable data={property_type_dist} rowShading=true>
    <Column id=property_type />
    <Column id=count contentType=colorscale colorScale=info />
</DataTable>

*property_type* shows upper/lower casing discrepancies 

```property_type_lower_dist
select lower(property_type) pt, count(*) count
from properties.properties
group by lower(property_type)
```
<DataTable data={property_type_lower_dist} rowShading=true>
    <Column id=pt title="property_type (corrected)" />
    <Column id=count contentType=colorscale colorScale=info />
</DataTable>


## Business Logic & Reasonableness Check

### 7. Logical Checks

#### Logical Duplicates

```properties_same_property_name
select lower(property_name) property_name, count(*) duplicate_count
from properties.properties
group by lower(property_name) having count(*) > 1
```

**<Value data={properties_same_property_name} column=duplicate_count />** records for property_name ***<Value data={properties_same_property_name} column=property_name />***

*Robinson Plaza*
```properties_duplicate
select * 
from properties.properties 
where lower(property_name) in (
    select lower(property_name)
    from properties.properties
    group by lower(property_name) having count(*) > 1
)
```
<DataTable data={properties_duplicate} />

### 8. Reasonableness Checks

*extremely old properties*
```properties_year_built_vs_size
select year(current_date) - year_built age, total_sqft
from properties.properties
```
<ScatterPlot
    data={properties_year_built_vs_size}
    x=age
    y=total_sqft
/>