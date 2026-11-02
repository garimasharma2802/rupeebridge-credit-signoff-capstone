# RupeeBridge AI Credit-Score Sign-Off: Capstone

## Team and who did what

Garima Sharma completed all four Parts, the dashboard, the presentation and the repository on her own. The other team members did not respond or contribute.

Tools used: Excel for Parts 1 to 3, Power BI Desktop for Part 4, and Powerpoint for the presentation.

## Files in this repository

- `README.md`: all written answers, one section per task
- `data/applications.csv` and `data/field_inventory.csv`: the supplied files, unmodified
- `rupeebridge review.xlsx`: workings for Parts 1 to 3 (sheets Applications, Quality, Fields, Matrix, Dictionary, Fairness, plus Clean and Governance)
- `applications clean.csv`: output of Part 1 and input of the Part 4 dashboard
- `rupeebridge monitor.pbix`: the Power BI dashboard
- `RupeeBridge Presentation.pptx`: the presentation

The original columns A to I on the Applications sheet are unchanged. All calculations are in columns J onward and on the other sheets.

## Part 1: Is the application data good enough?

### Task 1.1 KPIs

Source: `applications.csv`, a sample of 120 applications (no date column, so the period is not stated).

| KPI | Formula | Source column | Result | Question it answers |
|---|---|---|---|---|
| Approval rate | approved applications divided by all applications | approved | 70.0% (84 of 120) | How many applicants get credit? |
| Completeness per field | filled cells divided by 120 | all 9 columns | 100% for 7 fields; monthly_income_inr 96.7%; credit_score 98.3% | Is the data complete enough to use? |
| Average credit score, approved vs rejected | average of cleaned credit score by approved | score_clean, approved | 718.7 vs 673.9 | Does the model approve higher-scoring applicants? |

### Task 1.2 Completeness

| Field | Filled | Blank | % missing |
|---|---|---|---|
| applicant_id | 120 | 0 | 0.0% |
| age | 120 | 0 | 0.0% |
| gender | 120 | 0 | 0.0% |
| city_tier | 120 | 0 | 0.0% |
| monthly_income_inr | 116 | 4 | 3.3% |
| existing_emi_inr | 120 | 0 | 0.0% |
| credit_score | 118 | 2 | 1.7% |
| call_intent_score | 120 | 0 | 0.0% |
| approved | 120 | 0 | 0.0% |

### Task 1.3 Outlier in income

Applicant A096 has a monthly income of 551,000 with z = 9.94. It is the only applicant with |z| above 3. The application form does not accept an income above 2,00,000, so this is a keying error (probably 55,100), not a real income. The mean income is 59,153 with it (116 valid values) and 54,876 without it.

### Task 1.4 Fixing missing and wrong values

I replaced the 4 blank incomes and the wrong income of A096 with the median of the valid incomes (53,200). I replaced the 2 blank credit scores with the median credit score (708.0). That is 5 of 120 incomes (4.2%) and 2 of 120 credit scores (1.7%), both under 5%. The cleaned mean income is 54,806 and the cleaned mean credit score is 705.2. The result is saved as `applications_clean.csv` with the new columns `income_clean` and `score_clean`.

### Task 1.5 Quality scorecard

| Dimension | Rating | Evidence |
|---|---|---|
| Accuracy | Poor | 1 impossible income: A096 = 551000, above the 200000 form limit |
| Completeness | Fair | 6 blank cells: 4 incomes (3.3%) and 2 credit scores (1.7%) |
| Consistency | Good | 0 duplicate IDs, 0 invalid gender values, 0 invalid city tiers, 0 EMIs above income |
| Timeliness | Fair | No application date column among the 9 fields, so the age of the data cannot be checked |
| Relevance | Fair | Credit score and call_intent_score are relevant, but gender is not needed to assess credit risk |


## Part 2: Is the data handled lawfully and safely?

### Task 2.1 Field inventory and minimisation

The model uses 7 of the 14 fields and does not use 7. Three fields are sensitive: religion, government_id_number and call_recording_transcript. 8 fields are not needed to assess credit risk (the 7 unused fields plus gender). Decisions: 8 drop, 2 mask, 4 keep.

| Field | Personal data | Sensitive | Needed by the model | Decision | Reason |
|---|---|---|---|---|---|
| customer_id | Yes | No | Yes | Mask | Needed as a join key; replace with a pseudonym |
| full_name | Yes | No | No | Drop | Not used and not needed to assess credit risk |
| mobile_number | Yes | No | No | Drop | Contact data is not needed for scoring |
| government_id_number | Yes | Yes | No | Drop | High-risk identifier and not used |
| date_of_birth | Yes | No | Yes | Mask | Only age is needed; store age, not the full date |
| home_address | Yes | No | No | Drop | Not used; city tier is enough |
| gender | Yes | No | No | Drop | Used but not needed; drives the gap in Part 3; keep only in audit data |
| religion | Yes | Yes | No | Drop | Sensitive and irrelevant to credit |
| employer_name | Yes | No | No | Drop | Not used |
| bank_account_number | Yes | No | No | Drop | Financial identifier and not used |
| monthly_income_inr | Yes | No | Yes | Keep | Core credit input |
| existing_emi_inr | Yes | No | Yes | Keep | Core credit input |
| credit_score | Yes | No | Yes | Keep | Core credit input |
| call_recording_transcript | Yes | Yes | Yes | Keep | Needed for call intent; restrict access, record consent, 24-month retention |

### Task 2.2 Privacy and security matrix

| | Low security need | High security need |
|---|---|---|
| High privacy need | None of the six assets | Customer KYC documents; credit-bureau scores; loan-application database; AI call-recording archive |
| Low privacy need | Public interest-rate card | Aggregated monthly approval statistics |

The aggregated statistics are not personal, so the privacy need is low. The security need is high because the figures must stay accurate (integrity).

| Asset | Recommendation |
|---|---|
| Customer KYC documents | Encrypt at rest, limit access to the KYC team, and delete after the retention period |
| Credit-bureau scores | Encrypt, restrict access by role, and share only with documented customer consent |
| Loan-application database | Apply role-based access and activity logging, and pseudonymise the data used for analytics |
| AI call-recording archive | Record consent, restrict playback to named staff, and delete after 24 months |
| Public interest-rate card | Publish openly with no special protection beyond normal change control |
| Aggregated monthly approval statistics | Share with the approval committee and protect against tampering so the figures stay accurate |

### Task 2.3 Privacy failure or security failure?

| Incident | Classification | Reason |
|---|---|---|
| (a) A hacker copies the application database | Security failure (and so a privacy breach) | An outsider got unauthorised access, so the protection failed and personal data was exposed as a result |
| (b) An employee sells call recordings to a marketing firm | Privacy failure | Customer data was used for a purpose the customers never agreed to |
| (c) A laptop with unencrypted bureau scores is stolen | Security failure | Unencrypted data was lost to an outsider, so a basic safeguard was missing |
| (d) Scores are shared with a partner bank without the customer's consent | Privacy failure | The data was shared without consent, even though nobody broke into the system |
| (e) Customer data collected for loans is used to target ads | Privacy failure | Data collected for one purpose was reused for another, with no breach of the system |

### Task 2.4 GDPR vs DPDP

| Topic | GDPR (customers in Germany) | DPDP Act (customers in India) |
|---|---|---|
| Scope of data | All personal data of people in the EU, on paper or digital; also reaches non-EU firms that offer services to people in the EU | Digital personal data (and digitised data) processed in India; also reaches processing outside India that serves people in India |
| Legal bases | Six bases: consent, contract, legal obligation, vital interests, public task and legitimate interests | Consent, plus a short list of legitimate uses such as legal obligations |
| Categories of personal data | Special categories (such as religion, health and biometric data) get stricter rules | One category of personal data; no separate special category in the Act |
| Data residency and transfers | Transfers outside the EU need safeguards, such as an adequacy decision or standard contractual clauses | Transfers are allowed unless the government restricts a country; sector rules (for example from the RBI) can still require data to stay in India |
| Individual rights | Access, correction, erasure (right to be forgotten), portability, objection, and limits on solely automated decisions | Access, correction, erasure, grievance redressal and nomination; the Act has no right to portability |
| Enforcement | National regulators; fines up to 20 million euros or 4% of global annual turnover | Data Protection Board of India; penalties up to 250 crore rupees |
| Who it applies to at RupeeBridge | RupeeBridge's non-resident Indian customers living in Germany | RupeeBridge's customers in India, for their digital personal data |

Because RupeeBridge is an Indian firm processing the data in India, its German customers' data is likely to fall under both laws.

Three actions RupeeBridge must take:
1. Build consent management: ask for clear, recorded consent that customers can withdraw.
2. Minimise data: collect only the fields needed and set a retention period for each.
3. Set a lawful basis and a transfer safeguard, such as standard contractual clauses, for sending German customers' data to India.

## Part 3: Who owns the data, and is the model fair?

### Task 3.1 Governance or stewardship?

| Activity | Classification | Reason |
|---|---|---|
| (a) Cleaning duplicate and blank values from the application extract | Data stewardship | Hands-on technical work to keep the data accurate |
| (b) Deciding which fields the model may use | Data governance | A policy decision about what data may be used |
| (c) Maintaining the description of each column | Data stewardship | Day-to-day upkeep of the documentation that describes the data |
| (d) Approving the retention period | Data governance | A rule set by those accountable for compliance, not a technical task |
| (e) Fixing a broken load of the bureau file | Data stewardship | Technical repair of the data pipeline |
| (f) Reviewing the model for bias and compliance | Data governance | Oversight of the model's fairness and legal compliance |

### Task 3.2 Data dictionary

| Column | Description | Data type | Observed range or values | Source | Quality rule |
|---|---|---|---|---|---|
| applicant_id | Unique reference for each loan application | Text | A001 to A120 (120 IDs) | Application system | Unique and never blank |
| age | Applicant's age in years | Whole number | 22 to 59 | Derived from date of birth (KYC form) | Whole number of at least 18 |
| gender | Applicant's gender | Text | F, M | KYC form | Only F or M |
| city_tier | Tier of the applicant's city | Text | Tier1, Tier2, Tier3 | Application data | Only Tier1, Tier2 or Tier3 |
| monthly_income_inr | Applicant's monthly income in rupees | Number | 25800 to 551000 (551000 is a keying error; all other values are 25800 to 124000); 4 blank | Application | Greater than 0 and no more than 200000 (the application form limit) |
| existing_emi_inr | Monthly instalments the applicant already pays, in rupees | Whole number | 200 to 39400 | Credit bureau | Zero or more, and below monthly income |
| credit_score | Credit-bureau score of the applicant | Number | 554 to 850; 2 blank | Credit bureau | Number within the bureau's valid range and never blank |
| call_intent_score | AI score derived from the verification call transcript | Whole number | 16 to 91 | Call centre and AI | Number from 0 to 100 and never blank |
| approved | Model decision: 1 means approved, 0 means rejected | Whole number | 0 or 1 (84 approved) | Model output | Only 0 or 1 |

Why metadata matters: metadata tells reviewers what each column means, where it came from and which values are valid, so the model's inputs can be trusted and checked.

### Task 3.3 Governance checks for the AI variable

`call_intent_score` in the sample: minimum 16, maximum 91, mean 59.89, and 0 missing values.

| Check type | Governance check |
|---|---|
| Quality | Check every month that scores are between 0 and 100 with none blank, and compare a sample of scores with a person's reading of the same calls |
| Bias and fairness | Compare the average score and the approval rate by gender and city tier every quarter, and investigate any group that is consistently scored lower |
| Compliance | Confirm that a recorded customer consent exists for call recording and AI processing, and delete transcripts after the 24-month retention period |

The average call-intent score is 60.7 for women and 58.9 for men, so this score does not explain the approval gap either.

### Task 3.4 Bias check on outcomes

By gender:

| Gender | Applications | Approval rate | Average credit score (cleaned) |
|---|---|---|---|
| F | 66 | 57.6% | 702.0 |
| M | 54 | 85.2% | 709.2 |

By city tier:

| City tier | Applications | Approval rate | Average credit score (cleaned) |
|---|---|---|---|
| Tier1 | 46 | 73.9% | 712.4 |
| Tier2 | 47 | 63.8% | 704.3 |
| Tier3 | 27 | 74.1% | 694.7 |

Equal-scoring groups do not receive equal outcomes: women average a credit score of 702.0 and men 709.2, but women are approved 57.6% of the time and men 85.2%. The gap is 27.6 percentage points, so demographic parity (equal positive outcomes across groups) does not hold. Credit quality does not explain the gap, because the scores are similar. By city tier the differences are much smaller. Recommendation: remove gender as a model input, re-test the approval rates by gender, and keep gender only in the audit data so fairness can still be monitored.

## Part 4: Monitoring dashboard, sign-off decision and presentation

### Task 4.1 Loading the data

I loaded `applications_clean.csv` into Power BI Desktop and checked that it has 120 rows. I set `applicant_id`, `gender` and `city_tier` as text, `age`, `existing_emi_inr`, `call_intent_score` and `approved` as whole numbers, and the income and credit score columns as numbers.

### Task 4.2 Cards and slicers

The dashboard has four cards (Applications as a distinct count of applicant_id, Approval Rate as the average of approved, Average Credit Score as the average of score_clean, and Average Income as the average of income_clean) and two slicers (gender as a list and city_tier as buttons).

| View | Applications | Approval Rate | Average Credit Score | Average Income |
|---|---|---|---|---|
| No filter | 120 | 70.0% | 705.2 | 54,806 |
| F selected | 66 | 57.6% | 702.0 | 56,708 |

![Full dashboard](dashboard%20full.png)

![Dashboard with F selected](dashboard%20female.png)

## Task 4.3 Fairness view: what the red cells mean

Red cells (approval rate below 60%): F x Tier2 (50.0%, 28 applications) and F x Tier3 (57.1%, 14 applications). F x Tier1 is 66.7%, but even this is below every male cell (81.8% to 92.3%). The female row is weak across all three tiers, so the gap found in Part 3 is not limited to one city tier. For the sign-off this means the model cannot be approved without conditions: gender must be removed and the model re-tested, and the matrix must be monitored every month.

## Task 4.4 Sign-off decision

**Decision: approve with conditions.**

Findings behind the decision:
- Part 1 (data quality): one income was a keying error (A096 = 551000, z = 9.94). With the blanks, 4.2% of income values and 1.7% of credit scores were replaced with medians. The defects are small and fixable.
- Part 2 (privacy and security): 7 of the 14 stored fields are not used by the model, including religion and government_id_number, which are sensitive or high-risk.
- Part 3 (governance and fairness): women are approved 57.6% of the time and men 85.2%, a gap of 27.6 percentage points, even though credit scores are similar (702.0 vs 709.2).
- Dashboard: F x Tier2 (50.0%) and F x Tier3 (57.1%) are below 60%.

Why not approve outright: the gender gap and the red cells show unequal outcomes. Why not reject: credit quality is similar across genders, and the data problems can be fixed and re-tested.

Conditions:
1. Remove gender as a model input and re-test. The approval gap must fall from 27.6 points to 5 points or less.
2. Block incomes above 200000 at the application form (A096 was 551000).
3. Fix missing values at source: 4 incomes (3.3%) and 2 credit scores (1.7%).
4. Drop or mask the 7 unused fields from the model dataset, including religion and government_id_number.
5. Record customer consent and apply the 24-month retention rule to call recordings.
6. Review the dashboard every month, with no matrix cell below 60% (today F x Tier2 and F x Tier3 are below it).

## Task 4.5 Remediation plan

| Action | Issue | Owner | Done when |
|---|---|---|---|
| Remove gender as a model input and re-test | 27.6-point approval gap | Governance | The re-test shows a gap of 5 points or less |
| Add an income limit check to the form | A096 was keyed as 551000 | Stewardship | The form rejects incomes above 200000 |
| Fix missing values at source | 4 blank incomes and 2 blank credit scores | Stewardship | The next extract has no blank incomes or credit scores |
| Drop or mask the 7 unused fields | 7 of 14 fields are not used, including religion and government ID | Governance | The model dataset holds only the fields it needs |
| Add consent records and a 24-month retention rule for call recordings | Call transcripts are used without a documented consent and retention rule | Governance | Every call has a consent record and transcripts older than 24 months are deleted |
| Add a recurring approval-rate check on the dashboard | The gender gap could return unnoticed | Stewardship | The dashboard is refreshed monthly and any cell below 60% is reported to the committee |

## Task 4.6 Presentation and repository

The presentation `RupeeBridge Presentation.pptx` has 8 slides: a cover, the problem, the approach, the data-quality findings, the privacy and security findings, the governance and fairness findings (with the approval-rate gap and the dashboard matrix), the sign-off recommendation with conditions and owners, and the risks and next steps. This repository is public and holds all the files listed at the top of this README.
