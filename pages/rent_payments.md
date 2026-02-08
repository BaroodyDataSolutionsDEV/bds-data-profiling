# Rent Payment Review

## rent_payments Table Structure

```rent_payments_summary
select * from rent_payments
```

| **Column** | **Data Type** | **Description** |
| ------ | --------- | ----------- |
| lease_id | number | foreign key to leases |
| payment_month | date | month of rent payment (yyyy-mm) |
| amount_due | number | amount of rent due |
| amount_paid | number | amount of rent paid |
| payment_status | text | categorical status of rent payment |


## rent_payment Data Profiling

### 1. Row Level Checks

#### Total number of rent_payment records
```rp_total_records
select count(*) total from rent_payments
```
- **<Value data={rp_total_records} column=total />** total rent_payment records.

## Null / Blanks summary
```rp_null_blank
select
    column_name,
    total_rows,
    null_or_blank_count,
    round(null_or_blank_count / total_rows, 2) null_or_blank_pct
from (
    select
        'lease_id' as column_name,
        count(*) as total_rows,
        sum(case when lease_id is null then 1 else 0 end) as null_or_blank_count
    from rent_payments
    union all
    select
        'payment_month' as column_name,
        count(*) as total_rows,
        sum(case when payment_month is null or payment_month = '' then 1 else 0 end) as null_or_blank_count
    from rent_payments
    union all
    select
        'amount_due' as column_name,
        count(*) as total_rows,
        sum(case when amount_due is null then 1 else 0 end) as null_or_blank_count
    from rent_payments
    union all
    select
        'amount_paid' as column_name,
        count(*) as total_rows,
        sum(case when amount_paid is null then 1 else 0 end) as null_or_blank_count
    from rent_payments
    union all
    select
        'payment_status' as column_name,
        count(*) as total_rows,
        sum(case when payment_status is null or payment_status = '' then 1 else 0 end) as null_or_blank_count
    from rent_payments
)
```
<DataTable data={rp_null_blank} rowNumbers=true rowShading=true>
    <Column id=column_name />
    <Column id=total_rows />
    <Column id=null_or_blank_count contentType=colorscale colorScale=info align=center />
    <Column id=null_or_blank_pct contentType=colorscale colorScale=info align=center />
</DataTable>



### 2. Primary Key Uniqueness Validation

#### Uniqueness of primary key

- Composite primary key made up of columns *lease_id*, *payment_month*, *payment_status*

```rp_unique_pk
select (select count(*) from (select distinct * from rent_payments)) total_records, count(distinct concat(cast(lease_id as int), '-', payment_month, '-', lower(payment_status))) composite_pk
from rent_payments
```

```rp_non_unique_pk
select concat(cast(lease_id as int), '-', payment_month, '-', lower(payment_status)) lease_month, count(*)
from (select distinct * from rent_payments)
group by concat(cast(lease_id as int), '-', payment_month, '-', lower(payment_status)) having count(*) > 1
```

```rp_pk_2
select concat(cast(lease_id as int), '-', payment_month) lease_month, payment_status
from rent_payments
where concat(cast(lease_id as int), '-', payment_month, '-', lower(payment_status)) in (select lease_month from ${rp_non_unique_pk})
```
<DataTable data={rp_pk_2} groupBy='lease_month' rowNumber=true rowShading=true>
    <Column id=lease_month title="leaseid-payment_month" />
    <Column id=payment_status title="Payment Status (casing)" />
</DataTable>

Discrepancies with payment_status preventing uniqueness of primary key
- Upper/Lower casing of payment_status values creating duplicate records



### 3. Foreign key integrity checks

#### *lease_id* check against **leases.csv**

```rent_payments_lease_id_cross_check
select distinct lease_id, case when lease_id in (select lease_id from leases) then '<span style="color: green;">True</span>' else '<span style="color: red;">False</span>' end lease_id_cross_check
from rent_payments
```
<DataTable data={rent_payments_lease_id_cross_check} rowNumbers=true rowShading=true>
    <Column id=lease_id align=center/>
    <Column id=lease_id_cross_check contentType=html align=center />
</DataTable>

```rp_lease_id_cross_check_pct
select sum(case when lease_id in (select lease_id from leases) then 1 else 0 end) / count(*) pct
from rent_payments
```
<BigValue data={rp_lease_id_cross_check_pct} value=pct title="lease_id's in leases table" fmt='pct'/>

## Data Consistency Checks

### 4. Date Field Validation

#### Earliest & Latest date values

```rp_payment_month_as_date
select lease_id, cast(payment_month || '-01' as date) payment_month_date, amount_due, amount_paid, payment_status
from rent_payments
```

*payment_month*

```rp_min_max_payment_month
select 
    strftime(min(payment_month_date), '%Y-%m') min_payment_month
    ,strftime(max(payment_month_date), '%Y-%m') max_payment_month
from ${rp_payment_month_as_date} 
```
<BigValue data={rp_min_max_payment_month} value=min_payment_month title="Earliest payment_month" />
<BigValue data={rp_min_max_payment_month} value=max_payment_month title="Latest payment_month" />

#### Check for future payment_months
```rp_future_payment_month
select count(*) total_records
from ${rp_payment_month_as_date}
where payment_month_date > (date_trunc('month', current_date) + interval 1 month)
```

**<Value data={rp_future_payment_month} column=total_records />** rent_payment records with future payment_month


#### Earliest & Latest payment_month for each lease_id
```rp_payment_month_lease_id
select
    lease_id,
    strftime(min(payment_month_date), '%Y-%m') min_payment_month,
    strftime(max(payment_month_date), '%Y-%m') max_payment_month
from ${rp_payment_month_as_date}
group by lease_id
```
<DataTable data={rp_payment_month_lease_id} rowNumbers=true rowShading=true>
    <Column id=lease_id align=center />
    <Column id=min_payment_month title="Earliest payment_month" align=center />
    <Column id=max_payment_month title="Latest payment_month" align=center />
</DataTable>



### 5. Numeric Field Validation

```rp_numeric_min_max_avg
    select 'amount_due' column_name, min(amount_due) min, max(amount_due) max, mean(amount_due) avg, sum(case when amount_due = 0 or amount_due is null then 1 else 0 end) null_or_zero_count
    from rent_payments
    union all
    select 'amount_paid' column_name, min(amount_paid) min, max(amount_paid) max, mean(amount_paid) avg, sum(case when amount_paid = 0 or amount_paid is null then 1 else 0 end) null_or_zero_count
    from rent_payments
```
<DataTable data={rp_numeric_min_max_avg} rowNumbers=true rowShading=true>
    <Column id=column_name align=center />
    <Column id=min />
    <Column id=max />
    <Column id=avg />
    <Column id=null_or_zero_count />
</DataTable>

*amount_due*
```rp_amount_due_dist
select amount_due from rent_payments where amount_due is not null
```
<Histogram data={rp_amount_due_dist} x=amount_due />

*amount_paid*
```rp_amount_paid_dist
select amount_paid from rent_payments where amount_paid is not null
```
<Histogram data={rp_amount_paid_dist} x=amount_paid />


```rp_amount_due_paid
select lease_id, sum(amount_due) amount, 'due' category
from rent_payments
group by lease_id
union all
select lease_id, sum(amount_paid) amount, 'paid' category
from rent_payments
group by lease_id
```
<BarChart data={rp_amount_due_paid} x=lease_id y=amount series=category type=grouped />


### 6. Text Field Profiling

```rp_text_max_avg
select
    'payment_status' column_name,
    max(length(payment_status)) max_length,
    mean(length(payment_status)) avg_length,
    sum(case when payment_status = '' then 1 else 0 end) empty_string_count,
    sum(case when payment_status is null then 1 else 0 end) null_count
from rent_payments
```
<DataTable data={rp_text_max_avg} rowNumbers=true rowShading=true>
    <Column id=column_name align=center />
    <Column id=max_length />
    <Column id=avg_length />
    <Column id=empty_string_count />
    <Column id=null_count />
</DataTable>

```rp_payment_status_common
select 
    'payment_status' column_name,
    payment_status text_label,
    count(*) total
from rent_payments
group by payment_status
order by total desc
```
<DataTable data={rp_payment_status_common} groupBy=column_name rowNumbers=true rowShading=true>
    <Column id=text_label title="Text" />
    <Column id=total title="Occurrences" />
</DataTable>

*payment_status* has upper/lower casing discrepancies

```rp_payment_status_lower_dist
select lower(payment_status) ps, count(*) count
from rent_payments
group by lower(payment_status)
order by count desc
```
<DataTable data={rp_payment_status_lower_dist} rowShading=true>
    <Column id=ps title="payment_status (corrected)" />
    <Column id=count contentType=colorscale colorScale=info />
</DataTable>


## Business Logic & Reasonableness Check

### 7. Logical Checks

#### Logical Duplicates

```rp_duplicates
select lease_id, payment_month, lower(payment_status) payment_status_lower, count(*) duplicate_count
from rent_payments
group by lease_id, payment_month, lower(payment_status) having count(*) > 1
```

```rp_leases_dup_count
select count(distinct lease_id) leases
from ${rp_duplicates}
```

**<Value data={rp_leases_dup_count} column=leases />** *lease_id*'s with duplicate records

```rp_same_stats
select lease_id, count(*) dup_rec
from ${rp_duplicates}
group by lease_id
order by 2 desc
```

<DataTable data={rp_same_stats} rowNumbers=true rowShading=true>
    <Column id=lease_id />
    <Column id=dup_rec title="Duplicated Entries" />
</DataTable>

```rp_dup_lease_ids
select distinct lease_id
from ${rp_duplicates}
```
<Dropdown data={rp_dup_lease_ids} name=dup_lease_id value=lease_id title="Select duplicate lease_id" />

<Slider
    title='Record window'
    name='dup_rec_window'
    size=large
    step=1
    min=1
    max=50
    defaultValue=10
/>

***lease_id*={inputs.dup_lease_id.value} deeper dive**

**Viewing (up to) {inputs.dup_rec_window} records before & after duplicate entry**

```rp_l10_dup
select *, row_number() over(order by cast(payment_month || '-01' as date)) a_id from (select distinct * from ${rp_duplicates}) where lease_id = ${inputs.dup_lease_id.value}
```

```rp_l10_dup_anchor_window
with ordered as (
    select *, row_number() over(partition by lease_id order by cast(payment_month || '-01' as date)) as rn
    from rent_payments
),
anchors as (
    select min(rn) as anchor_rn, 
        a.lease_id,
        a.payment_month,
        b.payment_status_lower,
        row_number() over(order by cast(a.payment_month || '-01' as date)) a_id
    from ordered a
        inner join ${rp_l10_dup} b on a.lease_id = b.lease_id
            and a.payment_month = b.payment_month
            and lower(a.payment_status) = b.payment_status_lower 
    group by a.lease_id, a.payment_month, b.payment_status_lower
)
select o.*, a.anchor_rn, a.a_id
from ordered o
    inner join anchors a on o.lease_id = a.lease_id
        and o.rn between a.anchor_rn - ${inputs.dup_rec_window} and a.anchor_rn + ${inputs.dup_rec_window}
order by a.anchor_rn, o.rn
```

```rp_l10_dup_anchor_window_table
select 
    a.a_id, b.a_id,
    case when b.lease_id is not null and a.a_id = b.a_id then '<span style="color: blue;">' || a.payment_month || '</span>' else a.payment_month end payment_month,
    case when b.lease_id is not null and a.a_id = b.a_id then '<span style="color: blue;">' || a.amount_due || '</span>' else a.amount_due end amount_due,
    case when b.lease_id is not null and a.a_id = b.a_id then '<span style="color: blue;">' || a.amount_paid || '</span>' else a.amount_paid end amount_paid,
    case when b.lease_id is not null and a.a_id = b.a_id then '<span style="color: blue;">' || a.payment_status || '</span>' else a.payment_status end payment_status
from ${rp_l10_dup_anchor_window} a
    left join ${rp_l10_dup} b on a.lease_id = b.lease_id
        and a.payment_month = b.payment_month
        and lower(a.payment_status) = b.payment_status_lower
order by a.anchor_rn, a.rn
```

<DataTable data={rp_l10_dup_anchor_window_table} groupBy=a_id rowNumbers=true rowShading=true>
    <Column id=payment_month contentType=html align=center/>
    <Column id=amount_due contentType=html align=center />
    <Column id=amount_paid contentType=html align=center />
    <Column id=payment_status contentType=html align=center />
</DataTable>



#### payment_month before lease_start_date or after lease_end_date

```rp_payment_month_check
select 
    l.lease_id, 
    cast(strftime(l.lease_start_date, '%Y-%m') || '-01' as date) lease_start_month_date,
    cast(strftime(l.lease_end_date, '%Y-%m') || '-01' as date) lease_end_month_date,
    rp.payment_month_date
from ${rp_payment_month_as_date} rp
    inner join leases l on l.lease_id = rp.lease_id
```

```rp_payment_month_before_start_date
select count(*) total
from ${rp_payment_month_check}
where payment_month_date < lease_start_month_date
```

```rp_payment_month_after_end_date
select count(*) total
from ${rp_payment_month_check}
where payment_month_date > lease_end_month_date
```

There are:
    - <Value data={rp_payment_month_before_start_date} column=total /> rent_payment records with *payment_month* **before** *lease_start_date*
    - <Value data={rp_payment_month_after_end_date} column=total /> rent_payment records with *payment_month* **after** *lease_end_date*


#### Check if *amount_due* matches leases *monthlyRent*

```rp_amt_due_monthlyRent
select *
from rent_payments rp
    inner join leases l on rp.lease_id = l.lease_id
where rp.amount_due != l.monthlyRent
```

All *amount_due* numbers in **rent_payments** match *monthlyRent* from **leases**




### 8. Reasonableness Checks

#### *payment_status* follows *amount_paid* vs. *amount_due*

```rp_status_paid
select lease_id, amount_due, amount_paid, lower(payment_status) payment_status_lower,
    case 
        when amount_paid > amount_due then 'over'
        when amount_paid < amount_due then 'under'
        when amount_paid = amount_due then 'equal'
        when amount_paid is null or amount_due is null then 'null'
    end payment_coverage
from rent_payments
```

```rp_status_coverage_summary
select payment_status_lower, payment_coverage, count(*) total
from ${rp_status_paid}
group by payment_status_lower, payment_coverage
order by 1, 2
```

<DataTable data={rp_status_coverage_summary} groupBy=payment_status_lower rowShading=true>
    <Column id=payment_coverage title="Payment Coverage" />
    <Column id=total title="Occurrences" />
</DataTable>


```rp_payment_status_dd_options
select distinct payment_status_lower
from ${rp_status_paid}
```
<Dropdown data={rp_payment_status_dd_options} name=rp_status_dd value=payment_status_lower>
    <DropdownOption value="%" valueLabel="All" />
</Dropdown>

<Dropdown name=rp_coverage_dd>
    <DropdownOption value="%" valueLabel="All" />
    <DropdownOption value="over" />
    <DropdownOption value="under" />
    <DropdownOption value="equal" />
    <DropdownOption value="null" />
</Dropdown>

```sql rp_payment_coverage
    select *, 
        case 
            when payment_status_lower = 'late' then '<span style="color: yellow;">Late</span>'
            when payment_status_lower = 'missed' then '<span style="color: red;">Missed</span>' 
            when payment_status_lower = 'paid' then '<span style="color: green;">Paid</span>'
        end payment_status_html,
        case 
            when payment_coverage = 'over' then '<span style="color: yellow;">Over</span>'
            when payment_coverage = 'under' then '<span style="color: red;">Under</span>'
            when payment_coverage = 'equal' then '<span style="color: green;">Equal</span>'
            when payment_coverage = 'null' then '<span style="color: blue;">Null</span>'
        end payment_coverage_html, 
    from ${rp_status_paid}
    where payment_status_lower like '${inputs.rp_status_dd.value}'
        and payment_coverage like '${inputs.rp_coverage_dd.value}'
    order by lease_id, payment_status_lower, payment_coverage
```

<DataTable data={rp_payment_coverage} rowShading=true>
    <Column id=lease_id />
    <Column id=amount_due />
    <Column id=amount_paid />
    <Column id=payment_status_html title="Payment Status" contentType=html align=center />
    <Column id=payment_coverage_html title="Payment Coverage" contentType=html align=center/>
</DataTable>



#### **rent_payment** record exists for each month of active *lease_id*

```sql rp_active_lease_months
    select
        l.lease_id, l.lease_start_date, l.max_rp_month,
        date_trunc('month', d) as month_start
    from (select *, (select max(payment_month_date) from ${rp_payment_month_as_date}) max_rp_month from leases) l
        cross join generate_series(
            date_trunc('month', l.lease_start_date),
            date_trunc('month', l.max_rp_month),
            interval 1 month
        ) as t(d)
    order by lease_id, month_start
```

```sql rp_active_lease_months_check
    select a.lease_id, a.month_start, 
        case when b.payment_month_date is null then 'null' else strftime(b.payment_month_date, '%Y-%m-%d') end payment_month_date
    from ${rp_active_lease_months} a
        left join ${rp_payment_month_as_date} b on a.lease_id = b.lease_id
            and a.month_start = b.payment_month_date
    where b.payment_month_date is null
    order by a.lease_id, a.month_start 
```

```sql rp_active_lease_months_check_records
    select l.lease_id, l.lease_start_date, l.max_rp_month, l.month_start, 
        case 
            when rp.payment_month_date is null then 'null' 
            else strftime(rp.payment_month_date, '%Y-%m-%d') 
        end payment_month_date, 
        rp.amount_due, rp.amount_paid, rp.payment_status
    from ${rp_active_lease_months} l
        left join ${rp_payment_month_as_date} rp on  l.lease_id = rp.lease_id
            and l.month_start = rp.payment_month_date
    where l.lease_id in (select lease_id from ${rp_active_lease_months_check})
    order by l.lease_id, l.month_start
```

```sql rp_active_lease_months_missing_summary
select lease_id, count(*) active_lease_months_missing_rent_payment_data
from ${rp_active_lease_months_check_records}
where payment_month_date = 'null'
group by lease_id
order by 2 desc
```

<DataTable data={rp_active_lease_months_missing_summary} rowShading=true rowNumbers=true />

<Dropdown data={rp_active_lease_months_missing_summary} name=lease_months_missing_rp value=lease_id />

```sql rp_active_lease_months_missing_summary_dt_data
select lease_id, lease_start_date, max_rp_month, case when payment_month_date = 'null' then '-' else payment_month_date end payment_month_date, payment_status  
from ${rp_active_lease_months_check_records} where lease_id = ${inputs.lease_months_missing_rp.value} order by lease_id, month_start
```
<DataTable data={rp_active_lease_months_missing_summary_dt_data} rowShading=true />