# Tenants Review

## Tenants Table Structure

```properties_summary
select * from tenants
```


| **Column** | **Data Type** | **Description** |
| ------ | --------- | ----------- |
| tenant_id | number | unique id assigned to each tenant record |
| company_name | string | name of the tenant's company |
| industry | string | categorical description of the industry of the tenant's company  | 
| num_employees | number | number of employees at the tenant's company |


## Tenants Data Profiling

### 1. Row Level & Structure Checks
```tenants_total_records
select count(*) total from tenants
```
- **<Value data={tenants_total_records} column=total />** total tenant records.


## Null / Blanks summary
```tenants_null_blank
select
    column_name,
    total_rows,
    null_or_blank_count,
    round(null_or_blank_count / total_rows, 2) null_or_blank_pct
from (
    select
        'tenant_id' as column_name,
        count(*) as total_rows,
        sum(case when tenant_id is null then 1 else 0 end) as null_or_blank_count
    from tenants
    union all
    select
        'company_name' as column_name,
        count(*) as total_rows,
        sum(case when company_name is null or company_name = '' then 1 else 0 end) as null_or_blank_count
    from tenants
    union all
    select 
        'industry' as column_name,
        count(*) as total_rows,
        sum(case when industry is null or industry = '' then 1 else 0 end) as null_or_blank_count
    from tenants
    union all
    select
        'num_employees' as column_name,
        count(*) as total_rows,
        sum(case when num_employees is null then 1 else 0 end) as null_or_blank_count
    from tenants
)
```
<DataTable data={tenants_null_blank} rowNumbers=true rowShading=true>
    <Column id=column_name />
    <Column id=total_rows />
    <Column id=null_or_blank_count contentType=colorscale colorScale=info align=center/>
    <Column id=null_or_blank_pct contentType=colorscale colorScale=info align=center />
</DataTable>

### 2. Primary key uniqueness validation

#### Uniqueness of primary key
```tenants_unique_pk
select count(*) total_records, count(distinct tenant_id) unique_tenant_ids
from tenants.tenants
```
<Value data={tenants_unique_pk} column=total_records /> total property records with <Value data={tenants_unique_pk} column=unique_tenant_ids /> unique tenant_id values.

### 3. Foreign key integrity checks

No foreign keys in tenants table

## Data Consistency Checks

### 4. Date Field Validation

No date fields in tenants table

### 5. Numeric Field Validation

```tenants_numeric_min_max_avg
select
    'num_employees' column_name,
    min(num_employees) min,
    max(num_employees) max,
    mean(num_employees) avg,
    sum(case when num_employees = 0 or num_employees is null then 1 else 0 end) null_or_zero_count
from tenants.tenants
```
<DataTable data={tenants_numeric_min_max_avg} rowNumbers=true rowShading=true>
    <Column id=column_name align=center />
    <Column id=min />
    <Column id=max />
    <Column id=avg />
    <Column id=null_or_zero_count />
</DataTable>

*num_employees*
```num_employees_dist
select num_employees from tenants.tenants
```
<Histogram data={num_employees_dist} x=num_employees />

### 6. Text Field Validation

```tenants_text_max_avg
select
    'company_name' column_name,
    max(length(company_name)) max_length,
    mean(length(company_name)) avg_length,
    sum(case when company_name = '' then 1 else 0 end) empty_string_count,
    sum(case when company_name is null then 1 else 0 end) null_count
from tenants.tenants
union all
select
    'industry' column_name,
    max(length(industry)) max_length,
    mean(length(industry)) avg_length,
    sum(case when industry = '' then 1 else 0 end) empty_string_count,
    sum(case when industry is null then 1 else 0 end) null_count
from tenants.tenants
```
<DataTable data={tenants_text_max_avg} rowNumbers=true rowShading=true>
    <Column id=column_name align=center />
    <Column id=max_length />
    <Column id=avg_length />
    <Column id=empty_string_count />
    <Column id=null_count />
</DataTable>

```tenants_text_common
select
    'industry' column_name,
    industry text_label,
    count(*) total
from tenants.tenants
group by industry
```
<DataTable data={tenants_text_common} groupBy=column_name rowNumbers=true rowShading=true>
    <Column id=text_label title="Text" />
    <Column id=total title="Occurrences" />
</DataTable>


*industry*
```tenants_industry_dist
select industry, count(*) count
from tenants.tenants
group by industry
```
<DataTable data={tenants_industry_dist} rowShading=true>
    <Column id=industry />
    <Column id=count contentType=colorscale colorScale=info />
</DataTable>


## Business Logic & Reasonableness Check

### 7. Logical Checks

#### Logical Duplicates

```tenants_same_company_name
select lower(company_name) company_name, count(*) duplicate_count
from tenants.tenants
group by lower(company_name) having count(*) > 1
```

{#each tenants_same_company_name as same}

**<Value data={tenants_same_company_name} column=duplicate_count />** records for company_name
***<Value data={same} column=company_name />***

{/each}


*Harrell LLC*
```tenants_dup_1
select * 
from tenants.tenants
where lower(company_name) = 'harrell llc'
```
<DataTable data={tenants_dup_1} />

*Jones LLC*
```tenants_dup_2
select *
from tenants.tenants
where lower(company_name) = 'jones inc'
```
<DataTable data={tenants_dup_2} />


### 8. Reasonableness Checks

*extremely large companies*
```tenants_num_employees
select 
    'Number of employees' as name, 
    min(num_employees) min, 
    percentile_disc(0.25) within group (order by num_employees) intervalBottom,
    median(num_employees) midpoint, 
    percentile_disc(0.75) within group (order by num_employees) intervalTop,
    max(num_employees) max
from tenants.tenants
```
<BoxPlot 
    data={tenants_num_employees}
    name=name
    min=min
    intervalBottom=intervalBottom
    midpoint=midpoint
    intervalTop=intervalTop
    max=max
    swapXY=true
/>

*industry alignment with property_type*
```industry_property_type
select distinct
    t.industry,
    p.property_type
from tenants.tenants t
    inner join leases.leases l on t.tenant_id = l.tenant_id
    inner join properties.properties p on p.property_id = l.property_id 
```
<DataTable data={industry_property_type} groupBy=industry rowShading=true>
    <Column id=industry title="tenant - industry" />
    <Column id=property_type title="properties - property_type" />
</DataTable>