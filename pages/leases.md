# Leases Review

## Leases Table Structure

```leases_summary
select * from leases.leases
```

| **Column** | **Data Type** | **Description** |
| ------ | --------- | ----------- |
| lease_id | number | primary key, unique id assigned to each lease record |
| property_id | number | foreign key to properties.csv, id of the property involved in the lease |
| tenant_id | number | foreign key to tenants.csv, id of the tenant involved in the lease |
| lease_start_date | date | start date of the lease (yyyy-mm-dd) |
| lease_end_date | date | end date of the lease (yyyy-mm-dd) |
| leased_sqft | number | square footage leased |
| monthlyRent | number | monthly rent (in dollars) |
| notes | unstructured text | text input associated with the lease |


## Table Structure Checks

### 1. Row Level Checks

#### Total number of lease records
```leases_total_records
select count(*) total from leases.leases
```
- **<Value data={leases_total_records} column=total />** total lease records.


#### Active lease summary
```current_date
select strftime(current_date, '%a, %-d %B %Y') today
```

```leases_active
    select count(*) active
    from leases.leases
    where current_date between lease_start_date and lease_end_date
```

```leases_expired
    select count(*) expired
    from leases.leases
    where current_date not between lease_start_date and lease_end_date
```


- As of <Value data={current_date} column=today /> there are:

    <BigValue data={leases_active} value=active title="Active Lease Records" />
    <BigValue data={leases_expired} value=expired title="Expired Lease Records" />


#### Null / Blanks summary
```leases_null_blank
select
    column_name,
    total_rows,
    null_or_blank_count,
    round(null_or_blank_count / total_rows, 2) as null_or_blank_pct
from (
    select 
        'lease_id' as column_name,
        count(*) as total_rows,
        sum(case when lease_id is null then 1 else 0 end) as null_or_blank_count
    from leases.leases
    union all 
    select
        'property_id' as column_name,
        count(*) as total_records,
        sum(case when property_id is null then 1 else 0 end) as null_or_blank_count
    from leases.leases
    union all
    select
        'tenant_id' as column_name,
        count(*) as total_records,
        sum(case when tenant_id is null then 1 else 0 end) as null_or_blank_count
    from leases.leases
    union all
    select
        'lease_start_date' as column_name,
        count(*) as total_records,
        sum(case when lease_start_date is null then 1 else 0 end) as null_or_blank_count
    from leases.leases
    union all
    select
        'lease_end_date' as column_name,
        count(*) as total_records,
        sum(case when lease_end_date is null then 1 else 0 end) as null_or_blank_count
    from leases.leases
    union all
    select
        'leased_sqft' as column_name,
        count(*) as total_records,
        sum(case when leased_sqft is null then 1 else 0 end) as null_or_blank_count
    from leases.leases
    union all
    select
        'monthlyRent' as column_name,
        count(*) as total_records,
        sum(case when monthlyRent is null then 1 else 0 end) as null_or_blank_count
    from leases.leases
    union all
    select
        'notes' as column_name,
        count(*) as total_records,
        sum(case when notes is null or trim(notes) = '' then 1 else 0 end) as null_or_blank_count
    from leases.leases
) t
```
<DataTable data={leases_null_blank} rowNumbers=true rowShading=true>
    <Column id=column_name />
    <Column id=total_rows />
    <Column id=null_or_blank_count contentType=colorscale colorScale=info align=center/>
    <Column id=null_or_blank_pct contentType=colorscale colorScale=info align=center />
</DataTable>

### 2. Primary key uniquness validation

#### Uniqueness of primary key
```leases_unique_pk
select count(*) total_records, count(distinct lease_id) unique_lease_ids
from leases.leases
```
<Value data={leases_unique_pk} column=total_records /> total lease records with <Value data={leases_unique_pk} column=unique_lease_ids /> unique lease_id values. 

### 3. Foreign key integrity checks

#### *property_id* check against **properties.csv**

```leases_property_id_cross_check
select distinct property_id, case when property_id in (select property_id from properties.properties) then '<span style="color: green;">True</span>' else '<span style="color: red;">False</span>' end property_id_cross_check
from leases.leases
```
<DataTable data={leases_property_id_cross_check} rowNumbers=true rowShading=true>
    <Column id=property_id align=center/>
    <Column id=property_id_cross_check contentType=html align=center />
</DataTable>

#### *tenant_id* check against **tenants.csv**

```leases_tenant_id_cross_check
select distinct
    tenant_id,
    case 
        when tenant_id in (select tenant_id from tenants.tenants) then '<span style="color: green;">True</span>'
        else '<span style="color: red;">False</span>'
    end tenant_id_cross_check
from leases.leases 
```
<DataTable data={leases_tenant_id_cross_check} rowNumbers=true rowShading=true>
    <Column id=tenant_id align=center/>
    <Column id=tenant_id_cross_check contentType=html align=center />
</DataTable>

## Data Consistency Checks

### 4. Date Field Validation

#### Earliest & Latest date values

*lease_start_date*
```leases_min_max_lease_start_date
select min(lease_start_date) min_lease_start_date, max(lease_start_date)max_lease_start_date 
from leases.leases
```
<BigValue data={leases_min_max_lease_start_date} value=min_lease_start_date title="Earliest lease_start_date" />
<BigValue data={leases_min_max_lease_start_date} value=max_lease_start_date title="Latest lease_start_date" />

*lease_end_date*
```leases_min_max_lease_end_date
select min(lease_end_date) min_lease_end_date, max(lease_end_date) max_lease_end_date from leases.leases
```
<BigValue data={leases_min_max_lease_end_date} value=min_lease_end_date title="Earliest lease_end_date" />
<BigValue data={leases_min_max_lease_end_date} value=max_lease_end_date title="Latest lease_end_date" />

#### Check for future start dates

```leases_future_lease_start_date
select count(*) total_records
from leases.leases
where lease_start_date > current_date
```

**<Value data={leases_future_lease_start_date} column=total_records />** lease records with future leases_start_date


#### Check for lease_end_date before lease_start_date

```leases_lease_end_date_before_lease_start_date
select count(*) total_records
from leases.leases
where lease_end_date < lease_start_date
```

**<Value data={leases_lease_end_date_before_lease_start_date} column=total_records />** lease records with *lease_end_date* before *lease_start_date*


#### Lease duration distribution (@TODO prorably move to end)

```leases_duration
select datediff('day', lease_start_date, lease_end_date) lease_duration_days
from leases.leases
```
<Histogram data={leases_duration} x=lease_duration_days />


### 5. Numeric Field Validation

```leases_numeric_min_max_avg
select 'leased_sqft' column_name, min(leased_sqft) min, max(leased_sqft) max, mean(leased_sqft) avg, sum(case when leased_sqft = 0 or leased_sqft is null then 1 else 0 end) null_or_zero_count
from leases.leases
union all
select 'monthlyRent' column_name, min(monthlyRent) min, max(monthlyRent) max, mean(monthlyRent) avg, sum(case when monthlyRent = 0 or monthlyRent is null then 1 else 0 end) null_or_zero_count
from leases.leases
```
<DataTable data={leases_numeric_min_max_avg} rowNumbers=true rowShading=true>
    <Column id=column_name align=center />
    <Column id=min />
    <Column id=max />
    <Column id=avg />
    <Column id=null_or_zero_count />
</DataTable>

*leased_sqft*
```leased_sqft_distribution
select leased_sqft from leases.leases
```
<Histogram data={leased_sqft_distribution} x=leased_sqft />

*monthlyRent*
```monthlyRent_distribution
select monthlyRent from leases.leases where monthlyRent is not null
```
<Histogram data={monthlyRent_distribution} x=monthlyRent />


### 6. Text Field Validation

```leases_text_max_avg
select 'notes' column_name, max(length(notes)) max_length, mean(length(notes)) avg_length, sum(case when notes = '' then 1 else 0 end) empty_string_count, sum(case when notes is null then 1 else 0 end) null_count
from leases.leases
```

*notes*
<DataTable data={leases_text_max_avg} rowNumbers=true rowShading=true>
    <Column id=column_name align=center />
    <Column id=max_length />
    <Column id=avg_length />
    <Column id=empty_string_count />
    <Column id=null_count />
</DataTable>

```leases_text_common
select notes, count(*) total
from leases.leases
group by notes
order by 2 desc
limit 5
```

Most common text in *notes*
<DataTable data={leases_text_common} rowNumbers=true rowShading=true>
    <Column id=notes />
    <Column id=total />
</DataTable>

## Business Logic & Reasonableness Check

### 7. Logical Checks

#### Logical Duplicates

```leases_same_property_and_tenant
select property_id, tenant_id, count(*) as duplicate_count
from leases
group by property_id, tenant_id
having count(*) > 1
```

{#if leases_same_property_and_tenant.length !== 0}
<DataTable data={leases_same_property_and_tenant} rowShading=true>
    <Column id=property_id />
    <Column id=tenant_id />
    <Column id=duplicate_count />
</DataTable>
{:else }
    0 logical duplicates found between property_id and tenant_id
{/if}

### 8. Reasonableness Checks

*leased_sqft* vs. *monthlyRent*
- check for:
    - large sqft with very low rent
    - small sqft with extremely high rent
```leases_leased_sqft_vs_monthlyRent
select property_id, leased_sqft, monthlyRent
from leases.leases
``` 
<ScatterPlot
    data={leases_leased_sqft_vs_monthlyRent}
    x=leased_sqft
    y=monthlyRent
/>

*monthlyRent* vs. lease duration in days
- check for:
    - very long lease with unusually low rent 
```leases_monthlyRent_vs_lease_duration
select datediff('day', lease_start_date, lease_end_date) lease_duration_days, monthlyRent
from leases.leases
```
<ScatterPlot
    data={leases_monthlyRent_vs_lease_duration}
    x=lease_duration_days
    y=monthlyRent
/>