# Quick Start

Get started with SQLatte in 5 minutes! This guide walks you through your first natural language queries.

## Step 1: Verify Installation

Make sure SQLatte is running:

```bash
curl http://localhost:8000/health
```

If you see a healthy response, you're ready to go! If not, check the [Installation Guide](installation.md).

## Step 2: Access the Interface

SQLatte provides multiple interfaces:

=== "Widget Interface"

    Open your browser to:
    ```
    http://localhost:8000/widget.html
    ```
    
    This gives you the standalone widget interface for testing.

=== "Embedded Widget"

    Add SQLatte to your HTML page:
    
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>My App with SQLatte</title>
    </head>
    <body>
        <h1>Sales Dashboard</h1>
        
        <div id="sqlatte-widget"></div>
        
        <script src="http://0.0.0.0:8000/static/js/sqlatte-widget.js"></script>
        <script>
            SQLatte.init({
                container: 'sqlatte-widget',
                apiUrl: 'http://0.0.0.0:8000'
            });
        </script>
    </body>
    </html>
    ```

=== "Dashboard (if enabled)"

    If you have analytics enabled:
    ```
    http://localhost:8000/dashboard
    ```

## Step 3: Your First Query

Let's try some natural language queries!

### Basic Query

Type in the widget:
```
Show me all customers
```

SQLatte will:
1. Convert your question to SQL
2. Execute the query
3. Display results in a table
4. Show the generated SQL

### Filter Query

Try adding conditions:
```
Show me customers from New York who signed up this year
```

### Aggregate Query

Get some analytics:
```
What's the total revenue by month for the last 6 months?
```

### Join Query

Combine tables:
```
Show me top 10 products by revenue with their category names
```

## Step 4: Understanding Results

When you submit a query, SQLatte shows:

**Generated SQL**
: The SQL query that was created from your natural language question

**Results Table**
: Data returned from your database, with pagination if needed

**Insights** (optional)
: AI-generated observations about the data

**Actions**
: Save query, schedule it, or download results

## Step 5: Save and Schedule

### Save a Query

Click the **Save** button after running a query. Give it a name:
```
Monthly Revenue Report
```

### Schedule a Query (if enabled)

If scheduling is enabled:

1. Click **Schedule** after running a query
2. Set frequency: Daily, Weekly, Monthly
3. Choose time: `09:00`
4. Add email recipients: `team@company.com`
5. Click **Create Schedule**

You'll receive automated reports via email!

## Understanding the Barista

SQLatte's "barista personality" means:

**It remembers context**
: "Show me those customers again" works in follow-up queries

**It suggests insights**
: "I notice revenue dropped in March - want to see why?"

**It clarifies when needed**
: "Which table are you asking about - customers or orders?"

**It stays friendly**
: Conversational tone instead of technical jargon

## Common Query Patterns

### Time-based Queries

```
Show me sales from last month
Show me daily active users for the past week
What's the trend of signups over the last quarter?
```

### Aggregations

```
What's the average order value by customer segment?
Count how many orders each customer has made
Sum up revenue by product category
```

### Rankings

```
Top 10 customers by total spend
Bottom 5 products by units sold
Most active users this week
```

### Comparisons

```
Compare revenue between Q1 and Q2
Show growth rate month over month
Which region has the highest conversion rate?
```

## Query Tips

!!! tip "Be Specific"
    Instead of: "Show me data"
    
    Try: "Show me customers who made purchases in the last 30 days"

!!! tip "Use Time References"
    SQLatte understands:
    - "last month", "this year", "yesterday"
    - "past 7 days", "last quarter"
    - "between January and March"

!!! tip "Name Your Metrics"
    Instead of: "Calculate that thing"
    
    Try: "Calculate total revenue", "Count active users"

!!! tip "Reference Table Names"
    If your database has multiple tables:
    
    "Show me customers from the users table"

## Example Conversation

Here's a realistic conversation with SQLatte:

```
You: Show me our top customers

SQLatte: Here are the top 10 customers by total order value...
[Results displayed]

You: What about last month specifically?

SQLatte: Here are the top customers for last month...
[Results displayed]

You: Can you show me what they purchased?

SQLatte: Here are the purchases from those customers...
[Results displayed with joins]

You: Schedule this to run monthly and email it to me

SQLatte: I've scheduled this query to run on the 1st of each month
and email results to your@email.com
```

## Next Steps

Now that you've run your first queries:

1. **Explore Configuration** - Customize SQLatte's behavior in [Configuration Guide](../configuration/overview.md)
2**Set Up Scheduling** - Automate reports with [Scheduled Queries](../features/schedules.md)
3**Enable Analytics** - Track usage with [Analytics Dashboard](../features/analytics.md)
4**Customize AI Behavior** - Use [Runtime Prompts](../features/runtime-prompts.md)

## Troubleshooting First Queries

### "No tables found"

Your database connection might not have permissions to see tables.

**Solution**: Grant proper permissions or check your database configuration.

### "I don't understand that question"

The query might be too vague or reference unknown tables.

**Solution**: Be more specific about table names and columns.

### "Query timeout"

The generated SQL might be too complex or the database is slow.

**Solution**: Try narrowing your question or optimizing database indexes.

### Results look wrong

The AI might have misunderstood your schema.

**Solution**: 
- Check table and column names match what you asked
- Review the generated SQL
- Be more explicit about joins and filters

## Sample Questions by Database Type

=== "E-commerce"

    ```
    Show me top selling products this month
    What's the average cart value?
    Which customers haven't ordered in 90 days?
    Compare sales between categories
    Show daily revenue for the last 30 days
    ```

=== "SaaS Analytics"

    ```
    How many active users do we have?
    What's the churn rate this quarter?
    Show signup trends over time
    Which features are most used?
    Calculate monthly recurring revenue
    ```

=== "Marketing"

    ```
    What's the conversion rate by campaign?
    Show email open rates by segment
    Which channels drive the most traffic?
    Calculate cost per acquisition
    Compare performance month over month
    ```

=== "Operations"

    ```
    Show inventory levels by warehouse
    Which orders are delayed?
    Calculate on-time delivery rate
    Show production output by shift
    Identify bottlenecks in the process
    ```

## Getting Help

If you get stuck:

- Check the **generated SQL** to understand what query was created
- Try rephrasing your question
- Be more specific about table and column names
- Open an issue on [GitHub](https://github.com/osmanuygar/sqlatte/issues)

Happy querying! ☕
