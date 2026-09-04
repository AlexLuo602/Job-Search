# Active Recall Priority Queue

This dashboard dynamically calculates your next review date for each problem based on the status of your **most recent attempt**. It only shows problems that are due for review.

> [!INFO] Intervals
> - **Failed / Gave Up / Gave Up (Redo):** 1 Day
> - **Should Redo / Redo (Failed):** 3 Days
> - **Too Long / Unoptimal and their redo variants:** 5 Days
> - **Done / Redo (Done):** 14 Days

```dataviewjs
// 1. Define Intervals based on status
const intervals = {
    "Failed": 1,
    "Gave Up": 1,
    "Gave Up (Redo)": 1,
    "Should Redo": 3,
    "Redo (Failed)": 3,
    "Too Long": 5,
    "Unoptimal": 5,
    "Redo (Too Long)": 5,
    "Redo (Unoptimal)": 5,
    "Done": 14,
    "Redo (Done)": 14
};

// 2. Get all attempt files
let pages = dv.pages('"Job Search/Neetcode/02. Attempts"').where(p => p.attempt_date);

// 3. Group by question (the wikilink to the question)
let grouped = pages.groupBy(p => p.question);

// 4. Find most recent attempt per question and calculate due date
let queue = [];
let today = dv.date('today').startOf('day');

for (let group of grouped) {
    // Sort attempts by date descending to get the latest
    let attempts = group.rows.sort(p => p.attempt_date, 'desc');
    let latest = attempts[0];
    
    // Get status and calculate interval
    let status = latest.status || "Done";
    let interval = intervals[status] || 14; 
    
    // Calculate Next Review Date
    let attemptDateStr = String(latest.attempt_date).split('T')[0];
    let attemptDate = dv.date(attemptDateStr);
    if (!attemptDate) continue;
    
    let nextReview = attemptDate.plus({ days: interval });
    let daysOverdue = Math.floor(today.diff(nextReview, 'days').days);
    
    // Add to queue if Due (nextReview <= today)
    if (daysOverdue >= 0) {
        queue.push({
            question: String(latest.question).replace(/^"|"$/g, ''), // Strip any stray YAML quotes
            latestAttempt: latest.file.link,
            status: status,
            daysOverdue: daysOverdue,
            concepts: Array.isArray(latest.review_concepts) ? latest.review_concepts.join("<br>") : (latest.review_concepts || "")
        });
    }
}

// 5. Sort by most overdue first
queue.sort((a, b) => b.daysOverdue - a.daysOverdue);

// Visual Formatters
function formatStatus(s) {
    if (s.includes("Failed") || s.includes("Gave Up")) return `🔴 **${s}**`;
    if (s.includes("Should Redo")) return `🟠 **${s}**`;
    if (s.includes("Too Long") || s.includes("Unoptimal")) return `🟡 **${s}**`;
    return `🟢 **${s}**`;
}

function formatOverdue(d) {
    if (d === 0) return `🎯 **Due Today**`;
    if (d === 1) return `🔥 **1 day overdue**`;
    return `🔥 **${d} days overdue**`;
}

// 6. Render the dashboard table
dv.table(
    ["Problem to Review", "Status", "Urgency", "Concepts to Review"],
    queue.map(q => [
        q.latestAttempt, 
        formatStatus(q.status), 
        formatOverdue(q.daysOverdue),
        q.concepts
    ])
);
```
