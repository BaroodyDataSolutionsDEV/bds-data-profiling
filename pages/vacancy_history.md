# Vacancy History Review

## vacancy_history Table Structure

```sql vh_summary
select * from vacancy_history 
```


| **Column** | **Data Type** | **Description** |
| ------ | --------- | ----------- |
| property_id | number | foreign key to properties |
| month | date | month of rent payment (yyyy-mm) |
| vacancy_rate | number | percentage of vacant spaces for property |


## vacancy_history Data Profiling

### 1. Row Level Checks

#### Total number of vacancy_history records
```sql vh_total_records
select count(*) total from vacancy_history
```
- **<Value data={vh_total_records} column=total />** from vacancy_history records.

## Null / Blanks summary
```sql vh_null_blank
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
    from vacancy_history
    union all
    select
        'month' as column_name,
        count(*) as total_rows,
        sum(case when month is null or month = '' then 1 else 0 end) as null_or_blank_count
    from vacancy_history
    union all
    select
        'vacancy_rate' as column_name,
        count(*) as total_rows,
        sum(case when vacancy_rate is null or month = '' then 1 else 0 end) as null_or_blank_count
    from vacancy_history
)
```
<DataTable data={vh_null_blank} rowNumbers=true rowShading=true>
    <Column id=column_name />
    <Column id=total_rows />
    <Column id=null_or_blank_count contentType=colorscale colorScale=info align=center />
    <Column id=null_or_blank_pct contentType=colorscale colorScale=info align=center />
</DataTable>



### 2. Primary Key Uniqueness Validation

#### Uniqueness of primary key

- No real primary key
    - records are uniquely identified by composite primary key made up of columns *property_id*, *month* and *vacancy_rate*

```sql vh_unique_pk
select
    count(*) total_records,
    count(distinct concat(property_id, '-', month)) composite_pk
from vacancy_history
```

```sql vh_non_unique_pk
select concat(property_id, '-', month) property_month, count(*)
from vacancy_history
group by concat(property_id, '-', month) having count(*) > 1
```

```vh_pk_2
select concat(property_id, '-', month) property_month, vacancy_rate
from vacancy_history
where concat(property_id, '-', month) in (select property_month from ${vh_non_unique_pk})
```

Duplicate *property*-*month* composite values have different vacancy rates
    - indicates that all 3 columns are required to uniquely identify a record

<DataTable data={vh_pk_2} groupBy='property_month' rowNumber=true rowShading=true>
    <Column id=property_month title="leaseid-payment_month" />
    <Column id=vacancy_rate title="Vacancy Rate" fmt='pct' />
</DataTable>


### 3. Foreign key integrity checks

#### *property_id* check against **properties.csv**

```sql vh_property_id_cross_check
select distinct property_id, case when property_id in (select property_id from properties) then '<span style="color: green;">True</span>' else '<span style="color: red;">False</span>' end property_id_cross_check
from vacancy_history
```
<DataTable data={vh_property_id_cross_check} rowNumbers=true rowShading=true>
    <Column id=property_id align=center/>
    <Column id=property_id_cross_check contentType=html align=center />
</DataTable>

```sql vh_property_id_cross_check_pct
select sum(case when property_id in (select property_id from properties) then 1 else 0 end) / count(*) pct
from vacancy_history
```
<BigValue data={vh_property_id_cross_check_pct} value=pct title="property_id's in properties table" fmt='pct'/>

## Data Consistency Checks

### 4. Date Field Validation

#### Earliest & Latest date values

```sql vh_month_as_date
select property_id, cast(month || '-01' as date) month_as_date, vacancy_rate
from vacancy_history
```

*vacancy_history*
```sql vh_min_max_month
select
    strftime(min(month_as_date), '%Y-%m') min_month,
    strftime(max(month_as_date), '%Y-%m') max_month
from ${vh_month_as_date}
```
<BigValue data={vh_min_max_month} value=min_month title="Earliest month" />
<BigValue data={vh_min_max_month} value=max_month title="Latest month" />

#### Check for future months
```sql vh_future_months
select count(*) total_records
from ${vh_month_as_date}
where month_as_date > (date_trunc('month', current_date) + interval 1 month)
```

**<Value data={vh_future_months} column=total_records />** **vacancy_history** records with future *month*

#### Earliest & Latest month for each property_id
```sql vh_month_property_id
select
    property_id,
    strftime(min(month_as_date), '%Y-%m') min_month,
    strftime(max(month_as_date), '%Y-%m') max_month
from ${vh_month_as_date}
group by property_id
```
<DataTable data={vh_month_property_id} rowNumbers=true rowShading=true>
    <Column id=property_id align=center />
    <Column id=min_month title="Earliest month" align=center />
    <Column id=max_month title="Latest month" align=center />
</DataTable>


### 5. Numeric Field Validation

```sql vh_numeric_min_max_avg
    select 
        'vacancy_rate' column_name, 
        min(vacancy_rate) min, 
        max(vacancy_rate) max, 
        mean(vacancy_rate) avg, 
        sum(case when vacancy_rate is null then 1 else 0 end) null_count,
        sum(case when vacancy_rate = 0 then 1 else 0 end) zero_count,
        sum(case when vacancy_rate = 1 then 1 else 0 end) one_count
    from vacancy_history
```
<DataTable data={vh_numeric_min_max_avg} rowNumbers=true rowShading=true>
    <Column id=column_name align=center />
    <Column id=min />
    <Column id=max />
    <Column id=avg />
    <Column id=null_count />
    <Column id=zero_count />
    <Column id=one_count />
</DataTable>

*vacancy_rate*
```vh_vacancy_rate_dist
select vacancy_rate from vacancy_history
```
<Histogram data={vh_vacancy_rate_dist} x=vacancy_rate />

<Dropdown
    data={vh_month_as_date} 
    name=prop_multi
    value=property_id
    multiple=true
/>

```sql linechart_vh_pid_month_rate
select vh.*
from ${vh_month_as_date} vh
where vh.property_id in ${inputs.prop_multi.value}
```

```sql linechart_annotations
    select 
        l.property_id,
        cast(strftime(lease_start_date, '%Y-%m') || '-01' as date) lease_start_month,
        vh.vacancy_rate,
        strftime(lease_start_date, '%Y-%m') lease_label
    from ${linechart_vh_pid_month_rate} vh
        inner join leases l on vh.property_id = l.property_id and cast(strftime(l.lease_start_date, '%Y-%m') || '-01' as date) = vh.month_as_date
```

<LineChart 
    data={linechart_vh_pid_month_rate}
    x=month_as_date
    y=vacancy_rate
    yAxisTitle="Vacancy Rate by Month"
    series=property_id
    subtitle="Reference points indicate lease_start_date from a valid record in the leases table"
>
    <ReferencePoint data={linechart_annotations} x=lease_start_month y=vacancy_rate label=lease_label labelPosition=bottom color=info />
</LineChart>


### 6. Text Field Profiling

No text fields


## Business Logic & Reasonableness Check

### 7. Logical Checks

```sql vh_100
select sum(case when vacancy_rate > 1 then 1 else 0 end) *1.0 / count(*) pct
from vacancy_history
```

```sql vh_0
select sum(case when vacancy_rate < 0 then 1 else 0 end) *1.0 / count(*) pct
from vacancy_history
```

- **<Value data={vh_100} column=pct fmt=pct />** records with vacancy_rate greater than 100%
- **<Value data={vh_0} column=pct fmt=pct />** records with vacancy_rate less than 0%


### 8. Reasonableness Checks

*see interactive chart above for vacancy_history trend as leases become active*