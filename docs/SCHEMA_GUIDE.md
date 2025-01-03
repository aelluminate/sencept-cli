# Schema Guide

The schema is a JSON object that defines the structure and rules for generating synthetic data. Each key in the schema represents a column in the dataset, and its value is a configuration object that specifies how the data for that column should be generated.

###### For example schema(s), refer to the **[EXAMPLES](EXAMPLES.md)** file.

## Types

- "`string`": Generates text data.
- "`number`": Generates numeric data.
- "`date`": Generates date values.
- "`time`": Generates time values.
- "`boolean`": Generates boolean values (0 or 1).


## Properties

### "`type`"
Specifies the data type of the column.  

**Allowed Values**: `string`, `number`, `date`, `time`, `boolean`.

**Example**:
```json
"age": {
    "type": "number"
}
```

### "`value`"
Assigns a fixed value to the column. If specified, all other properties are ignored.

**Allowed Values**: All

**Example**:
```json
"country": {
    "type": "string",
    "value": "Philippines"
}
```
### "`choices`"
Specifies a list of possible values for the column. A random value is selected from this list.

**Applicable Types**: `string`, `number`, `boolean`.

**Example**:
```json
"sex": {
    "type": "string",
    "choices": ["M", "F", "O"]
}
```

### "`weight`"
Specifies how random choices are weighted. If set to "random", weights are generated using a Dirichlet distribution. Various algorithms can be used to generate weights, such as `exponential`, `power_law`, `beta`, `lognormal`, and `zipf`.

**Applicable Types**: `string`, `number` (when `choices` is used).

**Example**:
```json
"payment_method": {
    "type": "string",
    "choices": ["GCash", "Maya", "Credit Card"],
    "weight": {
        "balanced": true,
    }
}
```

#### "`algorithm`" 🆕

- "`exponential`": Generates weights using an exponential distribution.
- "`power_law`": Generates weights using a power law distribution.
- "`beta`": Generates weights using a beta distribution.
- `"lognormal"`: Generates weights using a lognormal distribution.
- "`zipf`": Generates weights using a Zipf distribution.
- "`dirichlet`" (default): Generates weights using a Dirichlet distribution.

**Example**:
```json
"age": {
    "type": "number",
    "choices": [18, 25, 30, 35, 40, 45, 50, 55, 60],
    "weight": {
        "balanced": false,
        "algorithm": "beta"
    }
}
```

### "`range`"
Specifies a range of numeric values. A random number is generated within this range.

**Applicable Types**: `number`.

**Properties**:
- `min`: The minimum value.
- `max`: The maximum value.

**Example**:
```json
"age": {
    "type": "number",
    "range": {
        "min": 18,
        "max": 60
    }
}
```

### "`format`"

Specifies a custom format for generating values. This can include prefix, suffix, or random insertions.

**Applicable Types**: `number`.

**Properties**:
- `position`: Where to place the characters (`prefix`, `suffix`, or `random`).
- `contains`: The characters to use (e.g., `*`,`#`).
- `count`: The number of characters to add.

**Example**:
```json
"user_id": {
    "type": "number",
    "format": [
        {
            "position": "prefix",
            "contains": "63**",
            "count": 1
        },
        {
            "position": "suffix",
            "contains": "**",
            "count": 1
        }
    ],
    "length": 6
}
```

### "`dates`"
Specifies a date range for generating date values.

**Applicable Types**: `date`.

**Properties**:
- `min`: The minimum date (format: YYYY-MM-DD).
- `max`: The maximum date (format: YYYY-MM-DD) or if not specified, the current date.

**Example**:
```json
"joined_date": {
    "type": "date",
    "dates": {
        "min": "2024-01-01",
    }
}
```

### "`dependency`"
Specifies that the value of this column depends on another column.

**Applicable Types**: `date`, `time`.

**Properties**:
- `field`: The column this field depends on.
- `offset`: A range of days or time to add/subtract from the dependent field.
    - `min`: The minimum offset.
    - `max`: The maximum offset.

**Example**:
```json
"released_date": {
    "type": "date",
    "dependency": {
        "field": "purchased_date",
        "offset": {
            "min": 1,
            "max": 7
        }
    }
}
```

### "`mapping`"
Maps the value of this column to another column's value using a dictionary.

**Applicable Types**: `number`.

**Properties**:
- `field`: The column to map from.
- `values`: A dictionary of key-value pairs for mapping.

**Example**:
```json
"tier_discount_percentage": {
    "type": "number",
    "mapping": {
        "field": "loyalty_tier",
        "values": {
            "1": 3,
            "2": 5,
            "3": 7,
            "4": 10
        }
    }
}
```

### "`condition`"
Specifies a condition for generating the value of this column.

**Properties**:
- `field`: The column to check the condition against.
- `value`: The value the field must match.
- `values`: A list of values the field must match.

**Example**:
```json
"loyalty_tier": {
    "type": "string",
    "choices": [1, 2, 3, 4],
    "condition": {
        "field": "loyalty_program_member",
        "value": 1
    }
}
```

### "`inputs`"
Specifies a list of columns whose values are used as inputs for an operation.

- **Applicable Types**: `number`.

**Example**:
```json
"total_discount": {
    "type": "number",
    "inputs": ["total_discount_percentage", "total_purchase"],
    "operation": "percentage"
}
```

### "`operation`"
Specifies an operation to perform on the input values.

**Applicable Types**: `number`.

**Allowed Values**:
- "`sum`": Sums the input values.
- "`subtract`": Subtracts the second input value from the first.
- "`percentage`": Calculates a percentage of the input values.

**Example**:
```json
"total_purchase_after_discount": {
    "type": "number",
    "inputs": ["total_purchase", "total_discount"],
    "operation": "subtract"
}
```

### "`calculation`" 🆕
Specifies a dynamic calculation for generating the value of this column.

**Applicable Types**: `number`.

**Properties**:
- "`field`": The column to use for the calculation.
- "`operation`": The operation to perform.
  - "`divide`": Divides the value by the field. (for now, only divide is supported)
- "`value`": The value to use in the calculation.

**Example**:
```json
"loyalty_points_redeemed": {
    "type": "number",
    "dependency": [
        {
            "field": "loyalty_program_member",
            "value": 1
        }
    ],
    "calculation": {
        "field": "total_purchase",
        "operation": "divide",
        "value": 1000
    }
}
```


### "`parent`"
Specifies that the value of this column is tied to another column (e.g., for consistency across rows).

**Example**:
```json
"age": {
    "type": "number",
    "parent": "user_id"
}
```

### "`unique`"
Ensures that the values in this column are unique.

**Applicable Types**: `string`, `number`.

Example:
```json
"user_id": {
    "type": "number",
    "unique": true
}
```

### "`alphanumeric`" 🆕
Specifies whether the generated value should be alphanumeric (letters and numbers).

**Applicable Types**: `number`.

**Default**: false

**Example**:
```json
"tracking_number": {
    "type": "number",
    "format": [
        {
            "position": "prefix",
            "contains": "TN",
            "count": 1
        }
    ],
    "length": 10,
    "alphanumeric": true
}
```

### "`case`" 🆕
Specifies the case format of the generated string or alphanumeric value.

**Applicable Types**: string, number (when alphanumeric is true).

**Allowed Values**:

- "`uppercase`": Converts the string to uppercase.
- "`lowercase`": Converts the string to lowercase.
- "`mixed`": Leaves the string as-is (default).


**Example**:
```json
"product_name": {
    "type": "string",
    "case": "uppercase"
}
```