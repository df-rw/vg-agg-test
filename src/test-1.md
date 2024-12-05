---
title: testing vg aggregate funcs
sql:
    randomTimeline: ./rand-xy.csv
---

```sql id=foobar
-- Add state and size field to our timeline coordinate data.
CREATE OR REPLACE TABLE data AS (
    WITH
        states AS (
            SELECT 0 AS id, 'ACT' AS state
            UNION ALL SELECT 1, 'NSW'
            UNION ALL SELECT 2, 'NT'
            UNION ALL SELECT 3, 'NZ'
            UNION ALL SELECT 4, 'QLD'
            UNION ALL SELECT 5, 'SA'
            UNION ALL SELECT 6, 'TAS'
            UNION ALL SELECT 7, 'VIC'
            UNION ALL SELECT 8, 'WA'
        ),
        sizes AS (
            SELECT 0 AS id, 2 AS r
            UNION ALL SELECT 1, 4
            UNION ALL SELECT 2, 6
            UNION ALL SELECT 3, 8
        )
    SELECT
        ts,
        x,
        y,
        states.state,
        sizes.r,
    FROM
        randomTimeline
    LEFT JOIN states ON states.id = floor((x+y) % 9)
    LEFT JOIN sizes ON sizes.id = floor((x+y) % 4)
    ORDER BY ts
)
```

```js
// State menu and filter.
const stateFilter = vg.Selection.intersect();
const menuState = vg.menu({
    label: 'state',
    from: 'data',
    column: 'state',
    as: stateFilter,
});

// Aggregate ranges and expression.
const aggregateRanges = [
    { label: 'none', value: [0, 0] },
    { label: '5 seconds', value: [-5, 0] },
    { label: '10 seconds', value: [-10, 0] },
    { label: '30 seconds', value: [-30, 0] },
];
const aggregateExpression = vg.Param.array(aggregateRanges[0].value)
const menuAggregate = vg.menu({
    label: 'aggregate',
    options: aggregateRanges,
    as: aggregateExpression,
});

// Plot with no filter or expression applied.
const plotNoFilter = vg.plot(
    vg.lineY(vg.from('data'), {
        x: 'ts',
        y: 'r',
    }),
    vg.width(width),
    vg.height(100),
);

// Plot with filter applied.
const plotWithFilter = vg.plot(
    vg.lineY(vg.from('data', {
        filterBy: stateFilter,
    }), {
        x: 'ts',
        y: 'r',
    }),
    vg.width(width),
    vg.height(100),
);

// Plot with expression applied.
const plotWithAggregateExpression = vg.plot(
    vg.lineY(vg.from('data'), {
        x: 'ts',
        y: vg.avg('r').orderby('ts').rows(aggregateExpression),
    }),
    vg.width(width),
    vg.height(100),
);

// Plot with filter and expression applied.
const plotWithFilterAndAggregateExpression = vg.plot(
    vg.lineY(vg.from('data', {
        orderBy: 'ts',
    }), {
        x: 'ts',
        y: vg.avg('r').where(vg.eq('state', stateFilter)).orderby('ts').rows(aggregateExpression),
    }),
    vg.width(width),
    vg.height(100),
);
```

```js
console.log(
   vg.from('data'),
    vg.avg('r').where(vg.eq('state', stateFilter)).orderby('ts').rows(aggregateExpression)
);
```

<div class="card">
    <h2>Controls</h2>
    ${menuState}
    ${menuAggregate}
</div>

<div class="card">
    <h2>No state filter or aggregate function</h2>
    ${plotNoFilter}
</div>

<div class="card">
    <h2>State filter only</h2>
    ${menuState}
    ${plotWithFilter}
</div>

<div class="card">
    <h2>Aggregate function only</h2>
    ${plotWithAggregateExpression}
</div>

<div class="card">
    <h2>With filter and aggregate function</h2>
    ${plotWithFilterAndAggregateExpression}
</div>
