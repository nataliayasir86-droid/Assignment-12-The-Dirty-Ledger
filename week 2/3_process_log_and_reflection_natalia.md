# 3_log_reflection.md — Process Log and Reflection

## Workflow Log

| Step | Process | Finding / Result | Resolution |
|---|---|---|---|
| 1 | Read raw data and check datatypes | 97 rows, 6 columns. `price` is recognized as an object/string. | Do not overwrite source CSV. Use a copy. |
| 2 | Evaluate nulls | One missing price (2026-01-01, Pulse C, City 2). | Do not impute. Leave as NaN. |
| 3 | Evaluate duplicates | Found one exact duplicate (Staple A, City 1, 99.5 on 2026-01-08). | Drop the duplicate but export it to a separate CSV. |
| 4 | Clean price column | Commas (`1,250`) and a blank space are crashing numeric conversion. | Strip commas, cast to float, let blank become NaN. |
| 5 | Resolve 1,250 outlier | The paired City 1 value is 103.0. There is a reliable 6.0 difference between cities. | Adjust to 109.0, write an audit note, and keep the original string. |
| 6 | Resolve 9,999 outlier | The paired City 2 value is 163.5. The 6.0 gap applies here as well. | Adjust to 157.5, write an audit note, and keep the original string. |
| 7 | Fix text formatting | `oil b` is uncapitalized. | Change to `Oil B` for consistency. |
| 8 | Analyze 2026-02-05 `oil b` (108.5) | 108.5 perfectly tracks the Staple A prices. Real Oil B is ~318. | Change item to Staple A and document the assumption. |
| 9 | Final validation | 96 observations with 96 unique date-item-city combinations. | Confirmed correct panel dimensions. |

## AI Prompt Log

AI was utilized to draft pandas code and review my logical decisions. I checked the AI's logic against my raw data before applying it.

| AI Proposal | Decision | Rationale |
|---|---|---|
| Standardize `oil b` capitalization | ACCEPT | Basic formatting cleanup. |
| Strip commas and convert price to float | ACCEPT | Necessary for math operations; original text is kept as backup. |
| Overwrite 1,250 to 109.0 | REVISE | Only accepted because of the adjacent time-series data and city gap. Added an audit trail. |
| Overwrite 9,999 to 157.5 | REVISE | Only accepted because of the adjacent time-series data and city gap. Added an audit trail. |
| Re-categorize row 42 to Staple A | REVISE | Makes sense numerically, but it is an assumption. Requires a documented note. |
| Drop the duplicate row | ACCEPT | Leaving it would skew any future averages or sums. |
| Impute the missing price | REJECT | There is no statistical or source justification to fabricate this number. |
| Automatically drop extreme outliers | REJECT | Outliers should be investigated to find the underlying error, not blindly deleted. |

## Final Reflection

My final workflow intentionally deviated from a fully automated AI cleaning script in three distinct ways. First, I refused to impute the missing price value because there is no underlying source truth to base it on. Second, rather than blindly dropping the massive price outliers, I cross-referenced them with the paired city values and adjacent weeks to reconstruct the likely correct values, carefully noting my logic. Finally, I treated the lowercase product label as a symptom of a larger data-entry error rather than just a capitalization mistake.

The most critical piece of evidence during this audit was the consistent six-unit price margin between the two cities. If I hadn't mapped out the panel dimensions, I would have easily missed the duplicate row and the misclassified product. Furthermore, if I had just let the AI fill the missing value to make the table look perfect, I would have introduced fabricated data into the set.

While the resulting dataset is much cleaner and usable, it is important to remember it is not verified official data. The source file explicitly states these are mock teaching figures, and my manual price adjustments remain documented assumptions rather than confirmed facts.