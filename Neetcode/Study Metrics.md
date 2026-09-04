# Study Metrics

[[Job Search/Neetcode/Starting Point|← Starting Point]] · [[Job Search/Neetcode/Review Queue|Review Queue]]

This dashboard uses the latest attempt for each problem. A problem needs review when its latest status is not `Done` or `Redo (Done)`.

```dataviewjs
const attempts = dv.pages('"Job Search/Neetcode/02. Attempts"')
    .where(page => page.question && page.attempt_date);

const completeStatuses = new Set(["Done", "Redo (Done)"]);

function asList(value) {
    if (value == null) return [];
    if (typeof value === "string") return [value];
    if (Array.isArray(value)) return value;
    try {
        return Array.from(value);
    } catch {
        return [String(value)];
    }
}

function dateValue(value) {
    const date = dv.date(value);
    return date ? date.toMillis() : 0;
}

const latestByQuestion = new Map();
for (const page of attempts) {
    const key = String(page.question);
    const current = latestByQuestion.get(key);
    if (!current || dateValue(page.attempt_date) > dateValue(current.attempt_date)) {
        latestByQuestion.set(key, page);
    }
}

const latestAttempts = [...latestByQuestion.values()];

function summarize(field) {
    const groups = new Map();

    for (const attempt of latestAttempts) {
        const values = asList(attempt[field]);
        for (const rawValue of values) {
            const value = String(rawValue).trim();
            if (!value) continue;

            if (!groups.has(value)) {
                groups.set(value, {
                    problems: 0,
                    needsReview: 0,
                    totalMinutes: 0,
                    timedAttempts: 0,
                    latest: null,
                });
            }

            const group = groups.get(value);
            group.problems += 1;
            if (!completeStatuses.has(String(attempt.status))) group.needsReview += 1;

            const minutes = Number(attempt.time_min);
            if (Number.isFinite(minutes)) {
                group.totalMinutes += minutes;
                group.timedAttempts += 1;
            }

            if (!group.latest || dateValue(attempt.attempt_date) > dateValue(group.latest.attempt_date)) {
                group.latest = attempt;
            }
        }
    }

    return [...groups.entries()]
        .map(([area, group]) => ({
            area,
            ...group,
            reviewRate: group.problems ? group.needsReview / group.problems : 0,
            averageMinutes: group.timedAttempts
                ? Math.round(group.totalMinutes / group.timedAttempts)
                : null,
        }))
        .sort((a, b) =>
            b.needsReview - a.needsReview ||
            b.reviewRate - a.reviewRate ||
            b.problems - a.problems ||
            a.area.localeCompare(b.area)
        );
}

function renderSummary(title, rows) {
    dv.header(2, title);
    if (!rows.length) {
        dv.paragraph("No data yet.");
        return;
    }

    dv.table(
        ["Area", "Problems", "Needs Review", "Review Rate", "Avg. Minutes", "Latest Attempt"],
        rows.map(row => [
            row.area,
            row.problems,
            row.needsReview,
            `${Math.round(row.reviewRate * 100)}%`,
            row.averageMinutes ?? "—",
            row.latest?.file?.link ?? "—",
        ])
    );
}

const currentNeedsReview = latestAttempts.filter(
    attempt => !completeStatuses.has(String(attempt.status))
);

dv.paragraph(
    `Tracking **${latestAttempts.length}** problems. ` +
    `**${currentNeedsReview.length}** currently need review.`
);

renderSummary("Weak Areas by Topic", summarize("topic"));
renderSummary("Weak Areas by Review Concept", summarize("review_concepts"));
```

