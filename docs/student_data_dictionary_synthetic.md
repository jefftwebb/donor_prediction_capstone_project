# Student Data Dictionary for the Synthetic PBS Utah Donor Data

## Purpose and authority

This student-facing dictionary describes the files emitted by the synthetic
donor generator, the conventions that differ from production data, and known
release limitations. No row represents a real person or gift.

Use `data_dictionary_unite_and_team_approach.md`,
and `soft_credits_data_dictionary.md` for production key, crediting, scope,
and field semantics. 

## Public file inventory

| File | Unit | Main key or link | Synthetic use |
|---|---|---|---|
| `constituents_w_memberships.csv` | One row per constituent | `Constituent ID` | Static attributes and current snapshots |
| `unite_payments.csv` | One row per payment | `payment_id`; `donor_id` links to constituent | FY2021 onward payment history |
| `team_approach_legacy_payments.csv` | One row per legacy payment | `ta_account_id`; populated `constituent_id_1` links to constituent | Pre-FY2021 history used for donor features |
| `campaign_codes.csv` | One row per marketing code | `Marketing Code` | Catalog for payment marketing codes and campaign names |
| `campaign_members.csv` | One row per solicitation | No unique row key; `constituent_id` and `marketing_code` are links | Additive solicitation history, including non-responders |
| `passport_engagement_by_genre.csv` | One row per constituent × fiscal year × genre | `constituent_id`, `fiscal_year`, `genre` | Additive, genre-level Passport engagement; covered years only |
| `soft_credits.csv` | One row per synthetic soft credit | `Credit ID`; `Soft Credit Constituent ID` links to constituent | Recognition credit for a hard-credited organization gift |
| `cultivation.csv` | One donor-year assignment row | `synthetic_id`, `fiscal_year` | Selection, contact, holdout, and program adoption |
| `officer_portfolios.csv` | One row per synthetic portfolio | `portfolio_id` | Officer, capacity, and rollout year |

Files with fields documented in PBS Utah's original dictionaries reuse those
column names and meanings. `passport_engagement_by_genre.csv`,
`cultivation.csv`, and `officer_portfolios.csv` are synthetic-only teaching
tables; their fields and synthetic conventions are documented below.

## Synthetic-only columns

### cultivation.csv

| Field | Meaning |
|---|---|
| `synthetic_id` | Fabricated constituent identifier matching `Constituent ID` |
| `fiscal_year` | Fiscal year in which cultivation was assigned |
| `portfolio_id` | Synthetic officer portfolio assignment |
| `selected` | Donor selected for possible cultivation |
| `contacted` | Selected donor assigned contact rather than holdout |
| `holdout` | Randomized selected donor withheld from contact |
| `program_adopted` | Portfolio had adopted the synthetic cultivation program |

### officer_portfolios.csv

| Field | Meaning |
|---|---|
| `portfolio_id` | Fabricated portfolio identifier |
| `officer_id` | Fabricated officer identifier |
| `capacity` | Annual number of donors the portfolio can select |
| `rollout_fy` | Fiscal year of synthetic program adoption; blank for never treated |

## Synthetic conventions

- Fiscal years end June 30. Unite output begins in FY2021; earlier behavioral
  history is represented in Team Approach.
- Output identifiers are fabricated. They preserve only the documented joins
  needed for the student analysis.
- Fiscal-year giving and the $1,200 crossing target use recognition credit.
  A routed organization gift belongs to the soft-credited constituent; do not
  add its hard and soft sides as two gifts.
- A donor's annual giving band is preserved during emission. Large gifts are
  represented only through the published bands and generated values; no rare
  or extreme real amount is copied.
- `campaign_members.csv` and `passport_engagement_by_genre.csv` are additive
  tables. They do not change any base-release row. They were generated from
  aggregate statistics and the released tables using a seed stream separate
  from the base release.

### campaign_members.csv

The column names follow the source campaign-members schema; the date and
response-field conventions below are synthetic exceptions.
`constituent_id` values are the released
synthetic `Constituent ID` values, and `marketing_code` values are the
released campaign-code catalog values. `campaign_start_date` is the first day
of the represented fiscal year.

That date is a synthetic fiscal-year marker, not an observed contact date or
the original campaign's start date. Do not use it to infer contact timing.

The table has no unique row identifier. More than one solicitation may have
the same constituent, marketing code, and fiscal year for non-responses;
responding keys occur exactly once. Its direct joins are
many-to-one: `constituent_id` joins `Constituent ID` in
`constituents_w_memberships.csv`, and `marketing_code` joins `Marketing Code`
in `campaign_codes.csv`. Every value in both columns has a matching released
row, and `campaign_name` is copied from that campaign-code catalog row.

`responded` is derived from an existence relationship. It is `TRUE` exactly
when the cleaned released Unite payments contain at least one payment with the same
constituent, marketing code, and fiscal year; `responded_date` is the earliest
matching payment date. To reproduce or use this link, first reduce payments
to one row per constituent, marketing code, and fiscal year, then join that
result to the solicitation table. Joining raw payment rows directly can
duplicate solicitations when several payments share the response key.
`response_type` is `Payment` for linked responses and blank otherwise;
`indirect_response` is always `FALSE`. These fields are synthetic conventions,
not modeled relationships supplied by the additions pack.

For this link, payment IDs are normalized and deduplicated (first row kept);
dates must parse, amounts must be positive, and constituent and marketing-code
keys must join the released catalogs. Use `donor_id`, `marketing_code`, and
the fiscal year of `credit_date` as the payment-side response key. Normalize
both sides as stripped uppercase strings, remove a trailing `.0`, and strip
leading zeros on numeric constituent IDs. `credit_date` is used by the
synthetic convention; the original Unite dictionary labels it Date of Pledge.

The pack supplies anonymized campaign-category shares, not a mapping to the
released marketing codes. The generator assigns codes to those categories
artificially: payment-used codes belong to `Other`, and unused catalog codes
are distributed across the categories. This is not a real category-to-code
relationship. All linked responses consequently fall in `Other`; code-specific
response comparisons and catalog attributes must not be interpreted as
pack-supported effects.

The pack's solicitation-count distributions describe constituent-years with
at least one solicitation. Coverage probabilities are estimated from annual
pack volume and the published band/sustainer group weights, then sampled.
For a constituent-year with `k` distinct linked response keys, the generator emits `k`
responding solicitations and draws non-responses from a negative binomial
with `k` successes and success probability `r_b`, where `r_b` is one minus the
pack's non-response share for the giving band. A response key is the combination
of constituent, marketing code and fiscal year. Several linked payments on
the same key supply only one success. These response-bearing years
can have solicitation counts above the pack's p95.

Linked keys are additionally thinned within giving-band/sustainer cells.
Each previously linked key is retained independently with probability
`q = min(1, b/a)`, where `a` is the mean number of previously linked distinct
codes across all released donor-years in the cell (including zero-linked
years), and `b` is the model-implied mean solicitation count times the pack's
band-level response share. The pack has no cell means: the mean here comes
from the existing rounded p5–p95 interpolation with capped tails, not a
measured PBS Utah mean. Pack solicitation distributions condition on having
solicitations, so their denominator differs from `a`. This is a synthetic
modeling convention, not proof of excess code use in PBS Utah's records.
Cells with no linked keys retain `q = 1`; band 0 describes non-donor years.
All payments on an unretained key receive no matching solicitation. The
negative binomial uses the retained k; payment-link rates therefore fall
below the original pack-based linkage rates in thinned cells.

Selected constituent-years with no linked responses retain the pack's
band/sustainer quantile draw, interpolating p5–p95 with tails capped at those
endpoints. Counts and category shares are never adjusted to a realized total.
Adding these zero-response years means the pooled response share need not
equal `r_b`. Do not treat the synthetic response proportions as calibrated
PBS Utah response probabilities. The model also need not reproduce the
pack's distribution conditional on having at least one solicitation.

Response shares, solicitation counts, and payment-link rates are model-implied
values and should not be interpreted as PBS Utah's actual solicitation
behavior.

Neither additive table carries a planted effect on giving. They are generated
after released giving outcomes are fixed. Same-year solicitation responses
reveal that year's payments: do not use them to predict that year's giving.

### passport_engagement_by_genre.csv

This synthetic-only table replaces event-level Passport rows with five fields:

| Field | Meaning |
|---|---|
| `constituent_id` | Released synthetic `Constituent ID` |
| `fiscal_year` | Fiscal year of the covered constituent-year |
| `genre` | PBS Utah Passport genre label, with rare genres collapsed to `Other` in the pack |
| `titles_viewed` | Synthetic count of distinct titles viewed in the genre |
| `mean_percent_watched` | Synthetic mean watched fraction for the genre, on a 0–1 scale (0.5 means 50%) |

There is one row per covered constituent-year and genre. A covered
constituent-year has at least one linked viewing in the observed Passport
definition. No title, episode, viewing event, date, timestamp, Passport
account ID, or consent interval is emitted. An absent row can mean no linked
account, no consent, or no viewing; the source data do not distinguish those
cases.

`constituent_id` has a many-to-one join to `Constituent ID` in
`constituents_w_memberships.csv`; every value has a matching released
constituent. The combination of `constituent_id`, `fiscal_year`, and `genre`
is unique. There is no direct Passport-to-payment key. Passport coverage and
engagement were generated using the constituent's recognition-credit giving
band in the same fiscal year, but the table adds no event-level or payment-
level relationship.

Passport coverage is weakly associated with next-year crossing, as in PBS
Utah's records. For eligible FY2021--FY2025 donor-years, coverage is sampled
conditional on same-year giving band and the already-generated next-year
crossing outcome. This plants the released raw association; it does not make
Passport coverage a cause of giving. Activation timing is not modeled. First
observed coverage can be derived from the generated history, but analyses
should use coverage, not activation. Engagement by genre remains conditional
on the same-year giving band only.

Annual giving includes valid positive Unite payments even when their marketing
code is missing or does not join the catalog, plus valid recipient soft
credits. Payment IDs and Credit IDs are deduplicated before summing. Marketing
code validity limits response linking, not the giving-band calculation.

Coverage for FY2026 and for non-eligible constituent-years is sampled by
fiscal year, giving band, and era. Within covered constituent-years, genre
selection preserves the published per-genre viewer
probabilities in expectation while requiring at least one genre. Genre
combinations use a maximum-entropy convention; co-viewing preferences and
cross-year persistence are not estimated. Titles viewed and watched fraction
are drawn separately by band and genre from the published quantiles, capped
at p5 and p95. The pack does not support an additional relationship between
these two measures. Genre counts span the pack's full observation period;
using them in FY2021–FY2026 assumes the band-conditioned mix is stable.
Genre probabilities are approximations rather than observed pack viewer
shares. Genre-row shares and engagement quantiles are pooled-period
comparisons, not year-matched validation. Retained viewing-event counts have
a different grain from this table and cannot validate its row count.

## Known issues

This section is the home for post-release notes about the synthetic data.

- Second one-time gifts occur more often than in PBS Utah's records.
- Recurring schedules default to twelve paid months. Unite has more payment
  rows than PBS Utah's records for the same years, after unusable rows are
  removed.
- Synthetic legacy history is fully linked for behavioral reconstruction.
  Extra unmatched rows are emitted as orphans, producing more legacy payment
  rows than PBS Utah's records for the same years, after unusable rows are
  removed.
- The synthetic system cutover is FY2021 rather than the May 2020 production
  cutover.
- Soft-credit designation IDs intentionally join the corresponding synthetic
  payment. This differs from the production identifier semantics in PBS
  Utah's soft-credit dictionary. The release volume gate covers FY2021 onward.
- University degree, employment, and university-wide legacy fields are not
  modeled. Blank values in those nullable fields mean unavailable, not zero.
- Effect sizes in these data are not PBS Utah's, and which predictors matter
  reflects how the data were generated.
- The CSV files can exceed Excel's worksheet row limit. Use Python, R, a
  database, or Power Query rather than opening and resaving the full files in
  Excel.
- Split train and test data by fiscal year or donor, as appropriate. Randomly
  splitting donor-years can place one donor's history on both sides and leak
  future information.

## Note for the synthetic release: soft credits

The original `soft_credits_data_dictionary.md` describes PBS Utah's records.
This section describes how the synthetic release differs and how to use soft
credits in the major-donor label.

### How this release differs from PBS Utah's records

- **Credit IDs are not all unique.** Some rows are exact duplicates.
- **Designation Detail IDs match designation IDs in the payment table.** In
  PBS Utah's records they do not. Do not write logic that depends on this
  match.
- **The organizations that received hard credits appear in the constituent and
  payment tables.** In PBS Utah's records, Hard Credit Org IDs appear in no
  other table.
- **Hard Credit Org Type has two values,** `DAF Sponsor` and `Other Sponsor`.
  PBS Utah's records use a finer classification of their own.
- **Soft credits begin in fiscal year 2021.** PBS Utah's begin in 1983.
- **Each donation is soft-credited to one person.** The release contains no
  donations credited to both spouses.
- **Unmatched recipients carry the placeholder ID `SYN-ORPHAN`.**
- **Duplicate and unmatched rows occur at lower rates** than in PBS Utah's
  records.

### Using soft credits in the major-donor label

- **Crediting rule.** A donor's fiscal-year giving is the sum of (a) positive
  payments in which the donor, an individual in the constituent table, is the
  payer and (b) positive soft credits assigned to the donor. Remove exact
  duplicate rows before summing. For revenue totals, count payments only;
  soft credits recognize giving, they do not add revenue.
- **Recipients and shared credits.** A recipient may be identified by a
  Constituent ID or by a Spouse Constituent ID. In PBS Utah's records, one
  donation may be soft-credited to both spouses; credit each spouse with their
  own soft credit, and keep spouses together when you split data for training
  and testing.
- **Make the rule a parameter.** PBS Utah's recognition rule will be applied
  when pipelines are run on real data. That includes which organization and
  gift types count, how repeated Credit IDs that differ are treated, and how
  shared credits are handled. Write your pipeline so these can be changed in
  one place.
- **Report what you remove.** Your pipeline should report how many rows each
  cleaning rule removes, by table and fiscal year, rather than removing them
  silently.
- **Known limitation.** The split between direct and soft-credit giving among
  threshold crossers has not been validated against PBS Utah's records. Do not
  draw conclusions about the role of soft credits in major giving.
