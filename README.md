# Conditional Data Workflow Framework

![Draft](https://img.shields.io/badge/status-draft-yellow.svg)

## Sample Scenario

```cdwf
CDWF "Get User Profile" {

  Description: "Get user profile"

  DataSources {
    SQLServer: "127.0.0.1:1433@NorthWindDb";
    Redis: "127.0.0.1:6379/1";
    API: "https://api.example.com/profile";
  }

  Criteria {
    UserId = "fe929721-7b25-4716-b87c-6cdf91919eb6" REQUIRED;
    Avatar = false OPTIONAL;
  }

  Variables {
    UserId = "fe929721-7b25-4716-b87c-6cdf91919eb6" AS "User profile id" type GUID;
    Avatar = false AS "User avatar" TYPE BOOL;
    RoleId = 1 AS "Role id" TYPE INTEGER;
  }

  Rules {
    @ Check redis cache for user
    IF Redis.Data EXISTS UserId 1 THEN PROCESS;
    ELSE SAVE UserId TO Redis.Data THEN PROCESS;
    AFTER 3 MINUTES PROCESS AND DELETE Redis.Data;
  }

  Cases {
    CASE 1: IF RowCount == 0 THEN SHOW "User not found";
    CASE 2: IF RowCount == 1 THEN SHOW User;
  }

  Transformations {
    USER_ID -> userId;
    @ Remove hypens from guid
    ROLE_ID -> roleId;
  }

  Output {
    Kafka -> "Users-Topic";
    DataWarehouse -> "Fabrikam DWH";
    PingPost --> "Ping to PingOMatic";
  }
}

```

## Structural Keywords

| **Keyword**         | **Description**                                                                                  |
|---------------------|------------------------------------------------------------------------------------------------|
| **CDWF**            | Defines the start of the Conditional Data Workflow Framework and the overall structure.         |
| **Description**     | Provides a general overview and purpose of the workflow.                                        |
| **DataSources**     | Specifies the sources from which data will be fetched (e.g., SQL Server, Redis, API).           |
| **Criteria**        | Lists the conditions and filters required for the workflow to execute.                          |
| **Variables**       | Defines the data fields to be used in the process along with their descriptions.                |
| **Rules**           | Specifies the special rules or constraints to be applied during data processing.                |
| **Cases**           | Outlines specific scenarios or steps that will execute under certain conditions.                |
| **Transformations** | Defines transformations or formatting operations to be applied to the data.                     |
| **Output**          | Indicates where and how the processed data will be sent (e.g., Kafka, Data Warehouse).          |


## Operators & Logical Keywords

| **Keyword**   | **Description**                                                                                         |
|---------------|--------------------------------------------------------------------------------------------------------|
| **IF**        | Specifies a condition that must be met for the following action to execute.                             |
| **THEN**      | Indicates the action to take if the preceding `IF` condition is true.                                   |
| **ELSE**      | Specifies the action to take if the `IF` condition is false.                                           |
| **BEFORE**    | Defines an action or condition that must occur prior to a specific event or time.                       |
| **AFTER**     | Defines an action or condition that must occur following a specific event or time.                      |
| **AND**       | Combines multiple conditions that must all be true for the action to execute.                           |
| **OR**        | Combines conditions where at least one must be true for the action to execute.                          |
| **NOT**       | Negates a condition, indicating that the action occurs when the condition is false.                     |
| **REQUIRED**  | Marks a condition or variable as mandatory for the workflow to proceed.                                 |
| **OPTIONAL**  | Marks a condition or variable as non-mandatory; it can be omitted without affecting the workflow.       |
| **DELETE**    | Removes specified data from a source or intermediary storage after processing.                          |
| **SHOW**      | Outputs specified information to the screen, typically for logging, debugging, or monitoring purposes.  |


## Data Types & Transformations

| **Data Type**   | **Description**                                                                                   |
|-----------------|---------------------------------------------------------------------------------------------------|
| **STRING**      | Represents a sequence of characters, typically used for text data.                                |
| **INTEGER**     | Represents whole numbers without decimal points.                                                  |
| **DECIMAL**     | Represents numbers with decimal points, used for precise calculations (e.g., financial data).     |
| **DATE**        | Represents a calendar date without time (e.g., `YYYY-MM-DD`).                                     |
| **TIME**        | Represents a specific time of day without a date (e.g., `HH:MM:SS`).                              |
| **DATETIME**    | Combines date and time into a single value (e.g., `YYYY-MM-DD HH:MM:SS`).                         |
| **GUID**        | Globally Unique Identifier, used to uniquely identify records across systems.                     |

### Transform Operator `(->)`

The Transform Operator `(->)` is used to map, rename, or convert data fields during processing. It helps in standardizing field names or applying transformations to the data.

```
SOURCE_FIELD -> TARGET_FIELD;
```

- **SOURCE_FIELD**: The original data field name.
- **TARGET_FIELD**: The new field name or the transformed representation.

#### Examples

##### Renaming Fields

```
POST_ID -> pid;
CATEGORY_ID -> cat_id;
```

- Here, `POST_ID` is renamed to `pid`, and `CATEGORY_ID` to `cat_id`


#### Applying Transformations

```
ORDER_DATE -> formattedOrderDate;
```

- This can imply that `ORDER_DATE` is being transformed into a formatted date string stored in `formattedOrderDate`

#### Chaining Transformations

```
TOTAL_AMOUNT -> formattedTotal -> finalAmount;
```

- In this scenario, `TOTAL_AMOUNT` is first formatted as `formattedTotal`, and then potentially further processed into `finalAmount`

## Comments

To create comments use `@` or `#`

```
Output {
    @ User list topic
    Kafka -> "Users-Topic";
    # Add user to data warehouse
    DataWarehouse -> "Fabrikam DWH";
    @ Create ping for the post
    PingPost --> "Ping to PingOMatic"; # Hate pingomatic
  }
```