# PBS Utah donor data: SYNTHETIC

These tables are synthetic. No row is a real person, gift, or ID. Table names, columns, keys, and relationships do not necessarily match PBS Utah's actual or current schema. Do not use this release to infer how
PBS Utah stores or organizes donor data.

## What's in the folder

- `tables/`: `constituents_w_memberships.csv`, `unite_payments.csv`, `soft_credits.csv`, `team_approach_legacy_payments.csv`, `campaign_codes.csv`, `campaign_members.csv`, `passport_engagement_by_genre.csv`, `cultivation.csv`, `officer_portfolios.csv`
- `docs/`:
  - **Student Data Dictionary (SYNTHETIC).** This lists the files, the synthetic conventions, and the known issues.
  - **Reference data dictionaries.** They explain source-field meanings but do not guarantee that the synthetic tables reproduce PBS Utah's table layout, column set, keys, or relationships.

## Downloading from GitHub

The largest tables use Git LFS (Large File Storage). Install Git LFS, run
`git lfs install` once, then clone the repository and download the tables:

```bash
git clone https://github.com/jefftwebb/donor_prediction_capstone_project.git
cd donor_prediction_capstone_project
git lfs pull
```

If you download a GitHub ZIP instead, the large CSVs may be small Git LFS
pointer files rather than data. Use the commands above to obtain the tables.

## Before you start

- **File size.** Several payment and solicitation tables have more than 1,048,576 rows. Excel cuts off the extra rows without warning, so use Python, R, or a database.
- **Fiscal years** run July through June; FY2026 is July 2025 to June 2026 and is complete in this release.
- **Validation.** Split training and test data by donor, keeping spouses together, or by time. Never split randomly by row: a donor's other years would sit on both sides and leak future information.
- **Crediting.** Soft credits recognize gifts for which an organization received the hard credit, including gifts from donor-advised funds. For revenue totals, count payments only: soft credits provide constituent recognition and must not be added as additional revenue.
- **Population.** The constituents table includes people who never gave, and may include organization records. Filter to the population your question needs.
- **Legacy (Team Approach) rows** can list up to seven constituent IDs for one household. If you unpivot those columns and then sum, you count the same gift several times. Some legacy rows link to no constituent, as in the real data.
- **Snapshot fields,** such as the sustainer flag, major-donor class, and age, describe the donor as of the extract date, not in past years. Using them as features for earlier years leaks the future.

## Known issues

Known issues are listed in the Student Data Dictionary. New ones will be added there. The data files themselves will not change.

## Additive tables

Solicitation history and Passport viewing summaries were added without changing the existing tables.

## Reporting a problem

Report suspected data problems to Jeff.
