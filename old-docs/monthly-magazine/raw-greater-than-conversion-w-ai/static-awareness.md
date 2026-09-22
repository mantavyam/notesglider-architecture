# Static Awareness

## OVERVIEW

<figure><img src="../../.gitbook/assets/Screenshot (416).png" alt=""><figcaption></figcaption></figure>

## FILES TO WORK WITH

<figure><img src="../../.gitbook/assets/Screenshot (418).png" alt=""><figcaption></figcaption></figure>

## STEP BY STEP APPROACH

### STEP 1: Extract Tables with Headline: "About" into a new <1.docx>

### STEP 2: Format Data with AI

#### We'll use 2 prompts to reach our expected output,

#### What we're doing?:

* Remove "About" from Headline
* Bind Data Couples in Bullet Points

{% hint style="info" %}
### PROMPT - 1
{% endhint %}

```
UNDERSTAND:
I'm giving you data in this format:
About Canara Bank
Founded
1906
HQ
Bengaluru
Tagline
Together We Can

DATA ANALYSIS:
First Line with the word "About" is the Headline

Following subsequent 2 Lines are each Data of Headline Topic which exist one after another (Couples)
For Example, "Tagline" and "Together We Can" is one data of Headline "Canara Bank",

WHAT TO DO?
Remove the Word "About" from Headline

Couples in Bulleted Points on new Line separated by "-"

Menton Headline at Top Line followed by Couples on next line like:

PRINT FORMAT : 
Canara Bank
Founded - 1906
HQ - Bengaluru
Tagline - Together we can

Do you understand?
```

{% hint style="danger" %}
AI is used as an assistant for speed and efficiency, It is your duty to manually verify the correctness of output without blindly copy pasting the response.
{% endhint %}

#### What we're doing?:

* Separate Formatted Data into respective Categories

{% hint style="info" %}
### PROMPT 2
{% endhint %}

```
UNDERSTAND:
Now I'll give you data in this format:
Madhya Pradesh:
Capital: Bhopal
CM: Mohan Yadav
Governor: Mangubhai C. Patel

Kyrgyzstan
Capital - Bishkek
Currency - Som
President - Sadyr Japarov

Group of 77:
Founded: 15 June 1964
Members: 135 member states
Chair of the Group of 77: Uganda

World Economic Forum:
Founded: 24 January 1971
HQ: Cologny, Switzerland
President: Børge Brende

NITI Aayog:
Founded: 1 January 2015
HQ: New Delhi
Chairman: Narendra Modi
CEO: B V R Subrahmanyam

Coal India Limited (CIL)
Founded - 1975
HQ - Kolkata, West Bengal
CMD - PM Prasad

Air India
Founded - 1953
HQ - New Delhi
CEO - Campbell Wilson

HDFC Bank
Founded - August 1994
HQ - Mumbai
MD & CEO - Sashidhar Jagdishan
Tagline - We Understand Your World

National Surveillance Programme for Aquatic Animal Diseases
Started in - 2013
Implemented by - Department of Fisheries
Objective - Strengthening farmer-based disease surveillance system, reporting and investigating disease cases, and providing scientific support to farmers.

M.S. Swaminathan Award
Launched in - 2004
Given by - Trust for Advancement of Agricultural Sciences
First Winner - Dr Norman E. Borlaug (2005)
2023 Winner - Dr Surinder (Suri) M. Sehgal

DATA ANALYSIS
Do a data analysis of each and categorize each into their repective groups.
These are the Groups in Which You'll Categorize:
INTERNATIONAL ORGANISATION
INDIAN GOVERNMENT ENTITIES
COMPANIES
FINANCE
NATIONS
STATES
AWARDS
GOVERNMENT SCHEMES OR PROGRAMMES

WHAT YOU'LL DO?
INTERNATIONAL ORGANISATION: Includes organisations that operates internationally.
INDIAN GOVERNMENT ENTITIES: Includes India's Government Ministries, Organisations and anything of india that operates locally in India.
COMPANIES: Includes companies from different fields that are properly business entities.
FINANCE: Includes Financial Services Companies or payment solutions or insurance agencies or banks.
NATIONS: Includes Full fledged nations recognised by United Nations.
STATES: Includes states of Indian sub-continent.
AWARDS: Includes any award given for a achievment, contribution or recognition.
GOVERNMENT SCHEMES OR PROGRAMMES: Includes Government run schemes or programmes at national level(INDIA)

ATTENTION:
Just Mention the data below their respective Categories, Don't Number them.
In Companies do not mention financial services companies, banks, insurance or paymemt solutions because there is a different category for them called FINANCE.
Also any Headline mentioning "Yojana", "Programme" or "Mission" will come under GOVERNMENT SCHEMES OR PROGRAMME

GIVEN BELOW IS A SAMPLE CATEGORIZATION FOR YOUR UNDERSTANDING:
INTERNATIONAL ORGANISATION
Group of 77:
Founded: 15 June 1964
Members: 135 member states
Chair of the Group of 77: Uganda
World Economic Forum:
Founded: 24 January 1971
HQ: Cologny, Switzerland
President: Børge Brende

INDIAN GOVERNMENT ENTITIES
NITI Aayog:
Founded: 1 January 2015
HQ: New Delhi
Chairman: Narendra Modi
CEO: B V R Subrahmanyam
Coal India Limited (CIL)
Founded - 1975
HQ - Kolkata, West Bengal
CMD - PM Prasad

COMPANIES
Air India
Founded - 1953
HQ - New Delhi
CEO - Campbell Wilson

FINANCE
HDFC Bank
Founded - August 1994
HQ - Mumbai
MD & CEO - Sashidhar Jagdishan
Tagline - We Understand Your World

NATIONS
Kyrgyzstan
Capital - Bishkek
Currency - Som
President - Sadyr Japarov

STATES
Madhya Pradesh:
Capital: Bhopal
CM: Mohan Yadav
Governor: Mangubhai C. Patel

AWARDS
M.S. Swaminathan Award
Launched in - 2004
Given by - Trust for Advancement of Agricultural Sciences
First Winner - Dr Norman E. Borlaug (2005)
2023 Winner - Dr Surinder (Suri) M. Sehgal

GOVERNMENT SCHEMES OR PROGRAMMES
National Surveillance Programme for Aquatic Animal Diseases
Started in - 2013
Implemented by - Department of Fisheries
Objective - Strengthening farmer-based disease surveillance system, reporting and investigating disease cases, and providing scientific support to farmers.

NOTE: Don't categorize this data, it is already done and is given for understanding.
Do you Understand what You've to do?
Are you ready to receive data for formatting as instructed above?
```

{% hint style="danger" %}
AI is used as an assistant for speed and efficiency, It is your duty to manually verify the correctness of output without blindly copy pasting the response.
{% endhint %}

### STEP 3: Group all identical categories together

### STEP 4: Update in Final Designs

***

### NEW DIRECTIONS:

````
# CONTEXT
You are an advanced data processing assistant specialized in data integrity, semantic categorization, and precise structural formatting. You will be provided with a raw, unstructured, or semi-structured list of static awareness data (e.g., countries, banks, government ministries, schemes, awards, companies). Your goal is to process this data, classify it into highly specific semantic groups, format it to a strict Markdown template, and run a thorough validation sequence.

# TASK
Analyze the provided raw static data, categorize each entry into its respective group, format each entry under its corresponding category, and provide a clear count/deduplication validation summary at the end.

# GUIDELINES
### 1. Classification Groups
Categorize the entities strictly into these groups (and create custom groups only if outliers do not fit):
* **INTERNATIONAL ORGANISATION**: Includes organisations that operate internationally.
* **INDIAN GOVERNMENT ENTITIES**: Includes India's Government Ministries, Organizations, and any Indian entities that operate locally.
* **COMPANIES**: Proper commercial business entities across various sectors. *(Do not include financial service providers, banks, insurance, or payment companies here).*
* **FINANCE**: Banks, financial services, payment solutions, rating agencies, stockbrokers, or insurance agencies.
* **NATIONS**: Full-fledged sovereign nations recognized by the United Nations.
* **STATES**: States and Union Territories of the Indian subcontinent.
* **AWARDS**: Awards given for achievement, contribution, or recognition.
* **GOVERNMENT SCHEMES OR PROGRAMMES**: Government-run schemes, programs, or missions at the national level (specifically including any headline containing "Yojana", "Programme", "Mission", or "Scheme").

### 2. Output Formatting
Each categorized group and entity must strictly follow this formatting style. Do not use numbers for lists:

```markdown
# **CATEGORY_NAME**

## **ENTITY_NAME**

* ATTRIBUTE_1 - VALUE_1
* ATTRIBUTE_2 - VALUE_2
* ATTRIBUTE_3 - VALUE_3
```

### 3. Data Integrity & Deduplication Rules
* Do not alter the values, names, or details of the attributes from the source.
* Convert raw backslash-escaped markdown formatting (e.g., `\-`) into clean, standard Markdown.
* If an entity is duplicate (present more than once with identical or near-identical details), consolidate it into a single entry under the correct category.

### 4. Validation Report Requirements
At the very end of your output, append a short validation section detailing:
* **Unique Headline/Entity Count Validation**: The total number of unique items found in the source, the number of processed items in your output, and an explicit mention of any duplicate items that were consolidated.
* **Attribute Integrity Note**: A single, concise note confirming that all individual attribute lines per item were cross-checked and match the source exactly without omissions.

# CONSTRAINTS
- Mandatorily Gather relevant and complete context to create a TODO List before performing a plan of action for your edits.
- Do proper structured analysis, take your time freely, think step by step, Let's go.
````

#### WATCH VIDEO GUIDE

{% embed url="https://drive.google.com/file/d/1uz4rb8eMOm48fhb6PbV7ZrORbn5DRs0u/view?usp=drive_link" %}
