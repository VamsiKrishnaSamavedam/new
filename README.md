# Enhanced Python + Oracle Job Search Application

**Course:** CS5334 Advanced Information Processing  
**Semester:** Spring 2026  
**Project:** Programming Project  
**Student Name:** Vamsi Krishna Samavedam, Venkata Sai Krishna Yakkanti  
**NetID:** xtd15, vqr9  

---

## Project URL

**Main Project URL:**

```text
http://newfirebird.cs.txstate.edu/~xtd15/demo/proc/unix-version/html/index-py.html
```

**Direct Job Search Form:**

```text
http://newfirebird.cs.txstate.edu/~xtd15/demo/proc/unix-version/html/job_search-py.html
```

---

## 1. Project Overview

This project enhances the existing class demo job search application implemented using **Python + Oracle**.

The original demo allowed users to search jobs using fields such as job type, job title, specialization, company, location, salary, and keyword. In the original version, some fields were handled as exact SQL filters. For example, if a user searched for a specific job type or company, only exact matches were returned. This could remove potentially useful results too early.

In this enhanced version, I modified the Python implementation so that the following features are handled using **rating-based search logic**:

- Job type rating
- Company rating
- Similar-company matching
- Keyword search

Exact matches receive the highest rating, while related or similar matches are still returned with rating penalties. This makes the search behavior more flexible and closer to a real information retrieval system.

---

## 2. Original Demo Behavior

The original Python + Oracle job search demo first built a SQL query based on the submitted form fields.

Some fields, such as region, state, and city, were not directly added to the SQL `WHERE` clause. Instead, they were evaluated later using a rating strategy.

Other fields, such as job type, job title, specialization, and company, were handled as exact SQL filters.

For example:

- Searching for job type `Regular` returned only jobs with job type `Regular`.
- Searching for company `Intel` returned only jobs from `Intel`.
- Related job types or similar companies were not returned.

The limitation of this approach is that relevant results can be filtered out before the rating logic is applied.

---

## 3. Implemented Features

| Feature | Status | Description |
|---|---|---|
| Job Type Rating | Implemented | Job type is no longer used as a strict SQL filter. It is now used during rating calculation. |
| Company Rating | Implemented | Company name is no longer used as a strict SQL filter. Exact company matches receive the highest rating. |
| Similar Company Search | Implemented | When a company is selected, related companies are also returned with rating penalties. |
| Keyword Search | Implemented | The keyword field is parsed and searched against job title, specialization, company, and other job table fields. |
| Database Table Changes | Not Required | No Oracle database table structure was modified. All enhancements were implemented in Python logic and data structures. |

---

## 4. Implementation Details

### 4.1 Job Type Rating Enhancement

In the original demo, `job_type` was added directly to the SQL `WHERE` clause. I changed this behavior so that job type is stored in:

```python
self.askedJobType
```

and evaluated later during rating.

This allows exact job type matches to receive the highest rating while related job types can still appear with a penalty.

Example rating idea:

| User Searches | Job Type Returned | Rating Behavior |
|---|---|---|
| Regular | Regular | Highest rating |
| Regular | Entry Level | Lower rating |
| Regular | Intern | Lower rating |
| Regular | Co-op | Lower rating |

This satisfies the requirement that job type should be used as part of the rating strategy instead of exact filtering.

---

### 4.2 Company Rating Enhancement

In the original demo, `company_name` was used as an exact SQL filter. I modified this logic so that company name is stored in:

```python
self.askedCompany
```

and evaluated later in the rating process.

Exact company matches receive no deduction and keep the highest rating.

For example, when the user searches for:

```text
company_name=Intel
```

jobs from Intel receive rating `100`.

---

### 4.3 Similar Company Search

I implemented a similar-company rating dictionary in `ematch_data_struct.py`.

This dictionary maps a selected company to related companies and their rating deductions.

Example:

```python
similarCompanies = {
    "intel": {
        "intel": 0,
        "amd": 10,
        "national semiconductor": 15,
        "texas instrument": 20,
        "motorola": 25
    }
}
```

When the user searches for Intel:

| Company Returned | Rating |
|---|---:|
| Intel | 100 |
| AMD | 90 |
| National Semiconductor | 85 |
| Texas Instrument | 80 |
| Motorola | 75 |

This confirms that exact company matches and similar company matches are both returned.

---

### 4.4 Keyword Search Enhancement

The front-end form already had a keyword input field:

```html
<input type="text" name="field_keyword">
```

However, the original Python code did not implement keyword search.

I added logic to parse the keyword from the query string and store it in:

```python
self.askedKeyword
```

The keyword is then evaluated during rating.

Keyword matching strategy:

| Match Location | Rating Behavior |
|---|---|
| Job title | No keyword deduction |
| Specialization | No keyword deduction |
| Company name | No keyword deduction |
| Other job table field | Small deduction |
| No keyword match | Job is not displayed |

Example:

```text
field_keyword=security
```

This returns jobs containing `security`, such as jobs with specialization `Network Security`.

---

## 5. Technical Approach

The main technique used in this project is **rating-based information retrieval**.

Instead of filtering out all non-exact matches in SQL, selected fields are stored and evaluated after candidate jobs are retrieved.

The final rating starts at:

```text
100
```

Each selected criterion can subtract a deduction.

| Match Type | Deduction |
|---|---:|
| Exact match | 0 |
| Similar or related match | Small deduction |
| Unrelated match | 100 |

If a job receives a deduction of `100` for an important selected criterion, it is not displayed.

The final rating combines:

- Location rating
- Job type rating
- Company rating
- Similar company rating
- Keyword rating
- Salary rating

---

## 6. Files Modified

### 6.1 `ematch.py`

Main project logic file.

Changes made:

- Modified `build_sql_query_stmt()`
  - `job_type` is no longer added as a strict SQL filter.
  - `company_name` is no longer added as a strict SQL filter.
  - `field_keyword` is parsed and stored for rating.
- Added:
  - `compute_job_type_rating()`
  - `compute_company_rating()`
  - `compute_keyword_rating()`
- Updated:
  - `compute_ratings()`

The updated `compute_ratings()` combines old and new rating logic.

---

### 6.2 `ematch_data_struct.py`

Data structure file used for rating dictionaries.

Changes made:

- Added `jobTypeSimilarity`
- Added `similarCompanies`

These dictionaries define rating deductions for related job types and similar companies.

---

### 6.3 `job_search-py.html`

Front-end job search form.

Changes made:

- Ensured the form submits to my CGI script:

```html
<form action="/~xtd15/cgi-bin/jobsearch.py">
```

- Kept the keyword field:

```html
<input type="text" name="field_keyword">
```

---

### 6.4 `jobsearch.py`

Controller CGI script.

Purpose:

- Reads submitted query string
- Normalizes salary values
- Connects to Oracle
- Creates an `ematch_class` object
- Calls the search, rating, sorting, and display logic

---

### 6.5 `index-py.html` and `title-py.html`

Navigation and frame files.

Changes made:

- Updated old professor/demo links from `~wp01` to my project path `~xtd15`.
- Ensured the main project page loads my own CGI and HTML files.

---

## 7. README / How to Run

### Main Project URL

Open this URL in a browser:

```text
http://newfirebird.cs.txstate.edu/~xtd15/demo/proc/unix-version/html/index-py.html
```

### Steps to Run

1. Open the main project URL.
2. Click **Job Search**.
3. Select search options such as:
   - Job Type
   - Job Title
   - Specialization
   - Company
   - Location
   - Salary
   - Keyword
4. Submit the form.
5. The result page displays matching jobs sorted by rating.

---

## 8. Example Test URLs

### 8.1 Job Type Rating Test

```text
http://newfirebird.cs.txstate.edu/~xtd15/cgi-bin/jobsearch.py?job_type=Regular&job_title=All&specialty=All&company_name=All&location_type=region&region=All&state=All&city=All&salary=Any&field_keyword=
```

Expected behavior:

- Regular jobs receive the highest rating.
- Related job types can appear with lower ratings.

---

### 8.2 Company Similarity Test

```text
http://newfirebird.cs.txstate.edu/~xtd15/cgi-bin/jobsearch.py?job_type=All&job_title=All&specialty=All&company_name=Intel&location_type=region&region=All&state=All&city=All&salary=Any&field_keyword=
```

Expected behavior:

| Company | Expected Rating |
|---|---:|
| Intel | 100 |
| AMD | 90 |
| National Semiconductor | 85 |
| Texas Instrument | 80 |
| Motorola | 75 |

---

### 8.3 Keyword Search Test

```text
http://newfirebird.cs.txstate.edu/~xtd15/cgi-bin/jobsearch.py?job_type=All&job_title=All&specialty=All&company_name=All&location_type=region&region=All&state=All&city=All&salary=Any&field_keyword=security
```

Expected behavior:

- Jobs containing the keyword `security` should be returned.
- Jobs without the keyword should not be displayed.

---

## 9. Testing and Results

### Test Case 1: Job Type Rating

**Input:**

```text
job_type=Regular
```

**Expected Result:**

Regular jobs should receive rating `100`. Related job types should still be allowed with rating penalties.

**Result:**

The system returned job results sorted by rating. Regular jobs appeared with rating `100`, confirming that exact job type matches receive the highest rating.

---

### Test Case 2: Company Similarity

**Input:**

```text
company_name=Intel
```

**Expected Result:**

Intel jobs should receive rating `100`. Similar companies should also appear with lower ratings.

**Result:**

The result page showed:

| Company | Rating |
|---|---:|
| Intel | 100 |
| AMD | 90 |
| National Semiconductor | 85 |
| Texas Instrument | 80 |
| Motorola | 75 |

This confirms that similar-company search works correctly.

---

### Test Case 3: Keyword Search

**Input:**

```text
field_keyword=security
```

**Expected Result:**

Jobs containing the keyword `security` should be returned. Jobs without the keyword should not be displayed.

**Result:**

The result page returned jobs containing `security`, such as jobs with specialization `Network Security`.

---

## 10. Screenshots to Include in Final Report

The following screenshots should be included in the final PDF report.

### Figure 1: Main Project Homepage


<img width="2880" height="1704" alt="image" src="https://github.com/user-attachments/assets/a86771f8-9ad1-4594-8415-40afc94a99f7" />


```text
http://newfirebird.cs.txstate.edu/~xtd15/demo/proc/unix-version/html/index-py.html
```

Caption:

Main project homepage loaded through index-py.html.

---

### Figure 2: Job Search Form

<img width="2880" height="1704" alt="image" src="https://github.com/user-attachments/assets/d5bc5573-ca72-42fc-898b-87aefa0da1b3" />


```text
http://newfirebird.cs.txstate.edu/~xtd15/demo/proc/unix-version/html/job_search-py.html
```

Caption:

Job search form with job type, company, salary, location, and keyword fields.


---

### Figure 3: Company Similarity Result

Intel company search result:

<img width="2880" height="1704" alt="image" src="https://github.com/user-attachments/assets/d499daff-45d5-4faa-94d3-9d74a102e56d" />
<img width="2880" height="1704" alt="image" src="https://github.com/user-attachments/assets/09d12d9d-5b84-4fa1-9b26-5117ac3347d7" />

```text
http://newfirebird.cs.txstate.edu/~xtd15/cgi-bin/jobsearch.py?job_type=All&job_title=All&specialty=All&field_keyword=&location_type=region&region=All&state=All&city=All&salary=Any&company_name=Intel
```

Caption:

Similar-company result for Intel. Intel receives rating 100, while AMD, National Semiconductor, Texas Instrument, and Motorola are returned with rating penalties.


---

### Figure 4: Keyword Search Result

Capture keyword search for `security`:

<img width="2880" height="1704" alt="image" src="https://github.com/user-attachments/assets/da54b4e6-31a4-4273-a297-6765e70d8375" />
<img width="2880" height="1704" alt="image" src="https://github.com/user-attachments/assets/0178ad98-e8a1-4a61-8d81-70b06d79a530" />

```text
http://newfirebird.cs.txstate.edu/~xtd15/cgi-bin/jobsearch.py?job_type=All&job_title=All&specialty=All&field_keyword=security&location_type=region&region=All&state=All&city=All&salary=Any&company_name=All
```

Caption:

Keyword search result for “security”. Jobs containing the keyword are returned.


---

### Figure 5: Job Type Rating Result

job type search:
<img width="2880" height="1704" alt="image" src="https://github.com/user-attachments/assets/b5abaa7e-1be3-4856-bf71-acb60b84e9b8" />
<img width="2880" height="1704" alt="image" src="https://github.com/user-attachments/assets/13c3dd86-5df9-463b-ba5d-370c648d93cf" />


```text
http://newfirebird.cs.txstate.edu/~xtd15/cgi-bin/jobsearch.py?job_type=Regular&job_title=All&specialty=All&company_name=All&location_type=region&region=All&state=All&city=All&salary=Any&field_keyword=
```

Caption:

Job type rating result for Regular jobs. Exact Regular jobs receive the highest rating, and related job types can appear with lower ratings.


---

## 11. Source Code Location

The source code is stored on the department Linux virtual machine under my account.

### CGI Source Files

```text
/home/xtd15/public_html/cgi-bin/
```

Important files:

```text
/home/xtd15/public_html/cgi-bin/jobsearch.py
/home/xtd15/public_html/cgi-bin/ematch.py
/home/xtd15/public_html/cgi-bin/ematch_data_struct.py
/home/xtd15/public_html/cgi-bin/db_config.py
```

### HTML Source Files

```text
/home/xtd15/public_html/demo/proc/unix-version/html/
```

Important files:

```text
/home/xtd15/public_html/demo/proc/unix-version/html/index-py.html
/home/xtd15/public_html/demo/proc/unix-version/html/job_search-py.html
/home/xtd15/public_html/demo/proc/unix-version/html/title-py.html
```

---

## 12. No Database Schema Modification

No Oracle database table structure was modified for this project.

All enhancements were implemented using:

- Python application logic
- Rating functions
- Dictionary-based similarity mappings

This keeps the implementation compatible with the original class demo database.

---

## 13. Conclusion

This project successfully enhanced the original Python + Oracle job search demo by adding rating-based job type search, company search, similar-company matching, and keyword search.

The enhanced system preserves the original demo structure while improving search flexibility. Instead of removing all non-exact matches through SQL filtering, the new system returns exact and related results using rating deductions.

The final implementation satisfies the required project goals without modifying the Oracle database schema.
