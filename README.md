WITH cohorts AS (
    SELECT
        user_id,
        DATE_TRUNC('month', signup_date) AS signup_month
    FROM users
),

user_activity AS (
    SELECT DISTINCT
        c.user_id,
        c.signup_month,
        DATE_TRUNC('month', a.activity_date) AS activity_month
    FROM cohorts c
    JOIN activity a
        ON c.user_id = a.user_id
),

cohort_retention AS (
    SELECT
        signup_month,
        COUNT(DISTINCT user_id) AS cohort_users,

        COUNT(DISTINCT CASE
            WHEN activity_month = signup_month
            THEN user_id
        END) AS month_0,

        COUNT(DISTINCT CASE
            WHEN activity_month = signup_month + INTERVAL '1 month'
            THEN user_id
        END) AS month_1,

        COUNT(DISTINCT CASE
            WHEN activity_month = signup_month + INTERVAL '2 months'
            THEN user_id
        END) AS month_2,

        COUNT(DISTINCT CASE
            WHEN activity_month = signup_month + INTERVAL '3 months'
            THEN user_id
        END) AS month_3

    FROM user_activity
    GROUP BY signup_month
)

SELECT
    signup_month,
    cohort_users,

    ROUND(month_0 * 100.0 / cohort_users, 2) AS month_0_retention,
    ROUND(month_1 * 100.0 / cohort_users, 2) AS month_1_retention,
    ROUND(month_2 * 100.0 / cohort_users, 2) AS month_2_retention,
    ROUND(month_3 * 100.0 / cohort_users, 2) AS month_3_retention

FROM cohort_retention
ORDER BY signup_month;
