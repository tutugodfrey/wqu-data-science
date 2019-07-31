

```python
%matplotlib inline
import matplotlib
import seaborn as sns
matplotlib.rcParams['savefig.dpi'] = 144
```


```python
from static_grader import grader
```

# PW Miniproject
## Introduction

The objective of this miniproject is to exercise your ability to use basic Python data structures, define functions, and control program flow. We will be using these concepts to perform some fundamental data wrangling tasks such as joining data sets together, splitting data into groups, and aggregating data into summary statistics.
**Please do not use `pandas` or `numpy` to answer these questions.**

We will be working with medical data from the British NHS on prescription drugs. Since this is real data, it contains many ambiguities that we will need to confront in our analysis. This is commonplace in data science, and is one of the lessons you will learn in this miniproject.

## Downloading the data

We first need to download the data we'll be using from Amazon S3:


```python
!ls
```

    dw.ipynb  in.ipynb  pw-Copy1.ipynb  pw-data  pw.ipynb



```bash
%%bash
mkdir pw-data
wget http://dataincubator-wqu.s3.amazonaws.com/pwdata/201701scripts_sample.json.gz -nc -P ./pw-data
wget http://dataincubator-wqu.s3.amazonaws.com/pwdata/practices.json.gz -nc -P ./pw-data
```

    mkdir: cannot create directory ‘pw-data’: File exists
    File ‘./pw-data/201701scripts_sample.json.gz’ already there; not retrieving.
    
    File ‘./pw-data/practices.json.gz’ already there; not retrieving.
    



```python
!ls
```

    dw.ipynb  in.ipynb  pw-Copy1.ipynb  pw-data  pw.ipynb


## Loading the data

The first step of the project is to read in the data. We will discuss reading and writing various kinds of files later in the course, but the code below should get you started.


```python
import gzip
import simplejson as json
```


```python
with gzip.open('./pw-data/201701scripts_sample.json.gz', 'rb') as f:
    scripts = json.load(f)

with gzip.open('./pw-data/practices.json.gz', 'rb') as f:
    practices = json.load(f)
    

```

This data set comes from Britain's National Health Service. The `scripts` variable is a list of prescriptions issued by NHS doctors. Each prescription is represented by a dictionary with various data fields: `'practice'`, `'bnf_code'`, `'bnf_name'`, `'quantity'`, `'items'`, `'nic'`, and `'act_cost'`. 


```python
print(len(scripts))
scripts[:2]
```

    382726





    [{'bnf_code': '0101010G0AAABAB',
      'items': 2,
      'practice': 'N81013',
      'bnf_name': 'Co-Magaldrox_Susp 195mg/220mg/5ml S/F',
      'nic': 5.98,
      'act_cost': 5.56,
      'quantity': 1000},
     {'bnf_code': '0101021B0AAAHAH',
      'items': 1,
      'practice': 'N81013',
      'bnf_name': 'Alginate_Raft-Forming Oral Susp S/F',
      'nic': 1.95,
      'act_cost': 1.82,
      'quantity': 500}]



A [glossary of terms](http://webarchive.nationalarchives.gov.uk/20180328130852tf_/http://content.digital.nhs.uk/media/10686/Download-glossary-of-terms-for-GP-prescribing---presentation-level/pdf/PLP_Presentation_Level_Glossary_April_2015.pdf/) and [FAQ](http://webarchive.nationalarchives.gov.uk/20180328130852tf_/http://content.digital.nhs.uk/media/10048/FAQs-Practice-Level-Prescribingpdf/pdf/PLP_FAQs_April_2015.pdf/) is available from the NHS regarding the data. Below we supply a data dictionary briefly describing what these fields mean.

| Data field |Description|
|:----------:|-----------|
|`'practice'`|Code designating the medical practice issuing the prescription|
|`'bnf_code'`|British National Formulary drug code|
|`'bnf_name'`|British National Formulary drug name|
|`'quantity'`|Number of capsules/quantity of liquid/grams of powder prescribed|
| `'items'`  |Number of refills (e.g. if `'quantity'` is 30 capsules, 3 `'items'` means 3 bottles of 30 capsules)|
|  `'nic'`   |Net ingredient cost|
|`'act_cost'`|Total cost including containers, fees, and discounts|

The `practices` variable is a list of member medical practices of the NHS. Each practice is represented by a dictionary containing identifying information for the medical practice. Most of the data fields are self-explanatory. Notice the values in the `'code'` field of `practices` match the values in the `'practice'` field of `scripts`.


```python
print(len(practices))
practices[:2]
```

    12020





    [{'code': 'A81001',
      'name': 'THE DENSHAM SURGERY',
      'addr_1': 'THE HEALTH CENTRE',
      'addr_2': 'LAWSON STREET',
      'borough': 'STOCKTON ON TEES',
      'village': 'CLEVELAND',
      'post_code': 'TS18 1HU'},
     {'code': 'A81002',
      'name': 'QUEENS PARK MEDICAL CENTRE',
      'addr_1': 'QUEENS PARK MEDICAL CTR',
      'addr_2': 'FARRER STREET',
      'borough': 'STOCKTON ON TEES',
      'village': 'CLEVELAND',
      'post_code': 'TS18 2AW'}]



In the following questions we will ask you to explore this data set. You may need to combine pieces of the data set together in order to answer some questions. Not every element of the data set will be used in answering the questions.

## Question 1: summary_statistics

Our beneficiary data (`scripts`) contains quantitative data on the number of items dispensed (`'items'`), the total quantity of item dispensed (`'quantity'`), the net cost of the ingredients (`'nic'`), and the actual cost to the patient (`'act_cost'`). Whenever working with a new data set, it can be useful to calculate summary statistics to develop a feeling for the volume and character of the data. This makes it easier to spot trends and significant features during further stages of analysis.

Calculate the sum, mean, standard deviation, and quartile statistics for each of these quantities. Format your results for each quantity as a list: `[sum, mean, standard deviation, 1st quartile, median, 3rd quartile]`. We'll create a `tuple` with these lists for each quantity as a final result.


```python
import statistics as stats

def quartile(n, pos):
    return pos/4*(n+1)

def describe(key):
    values_arr = sorted([script[key] for script in scripts])
    values_size = len(values_arr)
    total = sum(values_arr)
    avg = total/values_size
    s = stats.stdev(values_arr)
    q25 = values_arr[int(quartile(values_size, 1))]
    med = stats.median(values_arr)
    q75 = values_arr[int(quartile(values_size, 3))]

    return (total, avg, s, q25, med, q75)
```


```python
# dir(stats)
```


```python
describe('items')
```




    (4410054, 11.522744731217633, 33.11220959819492, 1, 3.0, 8)




```python
summary = [('items', describe('items')),
           ('quantity', describe('quantity')),
           ('nic', describe('nic')),
           ('act_cost', describe('act_cost'))]
```


```python
grader.score.pw__summary_statistics(summary)
```

    ==================
    Your score:  1.0
    ==================


## Question 2: most_common_item

Often we are not interested only in how the data is distributed in our entire data set, but within particular groups -- for example, how many items of each drug (i.e. `'bnf_name'`) were prescribed? Calculate the total items prescribed for each `'bnf_name'`. What is the most commonly prescribed `'bnf_name'` in our data?

To calculate this, we first need to split our data set into groups corresponding with the different values of `'bnf_name'`. Then we can sum the number of items dispensed within in each group. Finally we can find the largest sum.

We'll use `'bnf_name'` to construct our groups. You should have *5619* unique values for `'bnf_name'`.


```python
bnf_names = list(set([script['bnf_name'] for script in scripts]))
print(len(bnf_names))
print(bnf_names[:10])
assert(len(bnf_names) == 5619)
```

    5619
    ['LoFric Hydro Nelaton Male NonPVC 8-18(20', 'Psoriderm_Scalp Lot', 'Amoxicillin_Oral Susp Paed 125mg/1.25ml', 'Ebesque XL_Tab 300mg', 'Buscopan Cramps_Tab 10mg', 'Forticreme Complete_Dessert (Banana)', 'Aripiprazole_Tab 10mg', 'Oxycodone HCl_Tab 120mg M/R', 'Pioglitazone/Metformin HCl_Tab15mg/850mg', 'Convatec_Consecura L/Prof Closed Pch Opq']


We want to construct "groups" identified by `'bnf_name'`, where each group is a collection of prescriptions (i.e. dictionaries from `scripts`). We'll construct a dictionary called `groups`, using `bnf_names` as the keys. We'll represent a group with a `list`, since we can easily append new members to the group. To split our `scripts` into groups by `'bnf_name'`, we should iterate over `scripts`, appending prescription dictionaries to each group as we encounter them.


```python
groups = {name: [] for name in bnf_names}
# count = 0
# for group in groups:
#     if count == 4:
#         break
#     count += 1
#     print(group, groups[group])

for script in scripts:
    # INSERT ...
    groups[script['bnf_name']].append(script)
#     groups[script['bnf_name']].append(script['bnf_name'])
count = 0
for group in groups:
    if count == 4:
        break
    count += 1
    print(group, groups[group][:4])
# print(groups)
```

    LoFric Hydro Nelaton Male NonPVC 8-18(20 [{'bnf_code': '21020000969', 'items': 3, 'practice': 'L84033', 'bnf_name': 'LoFric Hydro Nelaton Male NonPVC 8-18(20', 'nic': 530.32, 'act_cost': 490.95, 'quantity': 14}, {'bnf_code': '21020000969', 'items': 1, 'practice': 'L84058', 'bnf_name': 'LoFric Hydro Nelaton Male NonPVC 8-18(20', 'nic': 189.4, 'act_cost': 175.34, 'quantity': 5}]
    Psoriderm_Scalp Lot [{'bnf_code': '1309000C0BDAAAE', 'items': 1, 'practice': 'P81020', 'bnf_name': 'Psoriderm_Scalp Lot', 'nic': 4.74, 'act_cost': 4.4, 'quantity': 250}, {'bnf_code': '1309000C0BDAAAE', 'items': 1, 'practice': 'Y04972', 'bnf_name': 'Psoriderm_Scalp Lot', 'nic': 4.74, 'act_cost': 4.4, 'quantity': 250}]
    Amoxicillin_Oral Susp Paed 125mg/1.25ml [{'bnf_code': '0501013B0AAAIAI', 'items': 1, 'practice': 'L84056', 'bnf_name': 'Amoxicillin_Oral Susp Paed 125mg/1.25ml', 'nic': 3.18, 'act_cost': 2.96, 'quantity': 20}, {'bnf_code': '0501013B0AAAIAI', 'items': 1, 'practice': 'Y05095', 'bnf_name': 'Amoxicillin_Oral Susp Paed 125mg/1.25ml', 'nic': 3.18, 'act_cost': 2.96, 'quantity': 20}]
    Ebesque XL_Tab 300mg [{'bnf_code': '0402010ABBFACAX', 'items': 1, 'practice': 'N81010', 'bnf_name': 'Ebesque XL_Tab 300mg', 'nic': 37.29, 'act_cost': 34.63, 'quantity': 28}, {'bnf_code': '0402010ABBFACAX', 'items': 4, 'practice': 'N81047', 'bnf_name': 'Ebesque XL_Tab 300mg', 'nic': 74.56, 'act_cost': 69.47, 'quantity': 56}, {'bnf_code': '0402010ABBFACAX', 'items': 5, 'practice': 'N83015', 'bnf_name': 'Ebesque XL_Tab 300mg', 'nic': 223.73, 'act_cost': 207.69, 'quantity': 168}, {'bnf_code': '0402010ABBFACAX', 'items': 2, 'practice': 'N83621', 'bnf_name': 'Ebesque XL_Tab 300mg', 'nic': 149.14, 'act_cost': 138.29, 'quantity': 112}]


Now that we've constructed our groups we should sum up `'items'` in each group and find the `'bnf_name'` with the largest sum. The result, `max_item`, should have the form `[(bnf_name, item total)]`, e.g. `[('Foobar', 2000)]`.


```python
def sum_items(group):
    total = 0
    for item in group:
        total += item["items"]
    return total

items_count = [(group, sum_items(groups[group])) for group in groups]
max_item = [("", 0)]
for item in items_count:
    if item[1] > max_item[0][1]:
        max_item[0] = item
print(max_item)
```

    [('Omeprazole_Cap E/C 20mg', 113826)]



```python
print(len(items_count))
print(items_count[0])

max(sorted(items_count))
```

    5619
    ('Co-Magaldrox_Susp 195mg/220mg/5ml S/F', 86)





    ('palmdoc (Reagent)_Strips', 6)



**TIP:** If you are getting an error from the grader below, please make sure your answer conforms to the correct format of `[(bnf_name, item total)]`.


```python
grader.score.pw__most_common_item(max_item)
```

    ==================
    Your score:  1.0
    ==================


**Challenge:** Write a function that constructs groups as we did above. The function should accept a list of dictionaries (e.g. `scripts` or `practices`) and a tuple of fields to `groupby` (e.g. `('bnf_name')` or `('bnf_name', 'post_code')`) and returns a dictionary of groups. The following questions will require you to aggregate data in groups, so this could be a useful function for the rest of the miniproject.


```python
def group_by_field(data, fields):
    groups = {}
    for item in data:
        group_by = ''
        if len(fields) == 1:
            group_by = item[fields[0]]
        else:
            field_list = []
            for field in fields:
                field_list.append(item[field])
            group_by = tuple(field_list)
            field_list = []
        if group_by in groups:
            groups[group_by].append(item)
        else:
            groups[group_by] = [item]
#     print(groups)
    return groups
```


```python
groups = group_by_field(scripts, ('bnf_name',))
# test_max_item = ...

def sum_items(group):
    total = 0
    for item in group:
        total += item["items"]
    return total

items_count = [(group, sum_items(groups[group])) for group in groups]
test_max_item = [("", 0)]
for item in items_count:
    if item[1] > test_max_item[0][1]:
        test_max_item[0] = item
print(test_max_item, max_item)
# count = 0
# for group in groups:
#     if count == 4:
#         break
#     print(group, len(groups[group]), '????????????????')
#     count += 1
assert test_max_item == max_item
```

    [('Omeprazole_Cap E/C 20mg', 113826)] [('Omeprazole_Cap E/C 20mg', 113826)]


## Question 3: postal_totals

Our data set is broken up among different files. This is typical for tabular data to reduce redundancy. Each table typically contains data about a particular type of event, processes, or physical object. Data on prescriptions and medical practices are in separate files in our case. If we want to find the total items prescribed in each postal code, we will have to _join_ our prescription data (`scripts`) to our clinic data (`practices`).

Find the total items prescribed in each postal code, representing the results as a list of tuples `(post code, total items prescribed)`. Sort your results ascending alphabetically by post code and take only results from the first 100 post codes. Only include post codes if there is at least one prescription from a practice in that post code.

**NOTE:** Some practices have multiple postal codes associated with them. Use the alphabetically first postal code.

We can join `scripts` and `practices` based on the fact that `'practice'` in `scripts` matches `'code'` in `practices'`. However, we must first deal with the repeated values of `'code'` in `practices`. We want the alphabetically first postal codes.


```python
print('SCRIPTS')
print(len(scripts))
scripts[:2]
```

    SCRIPTS
    382726





    [{'bnf_code': '0101010G0AAABAB',
      'items': 2,
      'practice': 'N81013',
      'bnf_name': 'Co-Magaldrox_Susp 195mg/220mg/5ml S/F',
      'nic': 5.98,
      'act_cost': 5.56,
      'quantity': 1000},
     {'bnf_code': '0101021B0AAAHAH',
      'items': 1,
      'practice': 'N81013',
      'bnf_name': 'Alginate_Raft-Forming Oral Susp S/F',
      'nic': 1.95,
      'act_cost': 1.82,
      'quantity': 500}]




```python
print('PRACTICES')
practices[:2]
```

    PRACTICES





    [{'code': 'A81001',
      'name': 'THE DENSHAM SURGERY',
      'addr_1': 'THE HEALTH CENTRE',
      'addr_2': 'LAWSON STREET',
      'borough': 'STOCKTON ON TEES',
      'village': 'CLEVELAND',
      'post_code': 'TS18 1HU'},
     {'code': 'A81002',
      'name': 'QUEENS PARK MEDICAL CENTRE',
      'addr_1': 'QUEENS PARK MEDICAL CTR',
      'addr_2': 'FARRER STREET',
      'borough': 'STOCKTON ON TEES',
      'village': 'CLEVELAND',
      'post_code': 'TS18 2AW'}]




```python
codes = [item['code'] for item in practices]

print (len(codes))
print(len(set(codes)))
```

    12020
    10843



```python
post_codes = [item['post_code'] for item in practices]

print (len(post_codes))
print(len(set(post_codes)))
```

    12020
    8306



```python
# scripts['practices'] == practices['code']

practice_postal = {}
for practice in practices:
    if practice['code'] in practice_postal:
        if practice['post_code'] < practice_postal[practice['code']]:
            practice_postal[practice['code']] = practice['post_code']
    else:
        practice_postal[practice['code']] = practice['post_code']

count = 0
for i in practice_postal:
    print(i, practice_postal[i])
    if count == 5:
        break
    count += 1
```

    A81001 TS18 1HU
    A81002 TS18 2AW
    A81003 TS25 1QU
    A81004 TS1 3BE
    A81005 TS14 7DJ
    A81006 TS18 2AT


**Challenge:** This is an aggregation of the practice data grouped by practice codes. Write an alternative implementation of the above cell using the `group_by_field` function you defined previously.


```python
practice_postal['K82019']
```




    'HP21 8TR'




```python
assert practice_postal['K82019'] == 'HP21 8TR'
```

**Challenge:** This is an aggregation of the practice data grouped by practice codes. Write an alternative implementation of the above cell using the `group_by_field` function you defined previously.


```python
practice_by_code = group_by_field(practices, ('code',))

# print(len(practices_by_postal))
practice_postal = {}
for practice_code in practice_by_code:
    practice_postal[practice_code] = practice_by_code[practice_code][0]['post_code']
    for practice in practice_by_code[practice_code]:
#         practice_postal[practice_code] = max(practice['post_code'], practice_postal[practice_code])
        if practice['post_code'] < practice_postal[practice_code]:
            practice_postal[practice_code] = practice['post_code']
            
count = 0
for i in practice_postal:
    print(i, practice_postal[i])
    if count == 5:
        break
    count += 1

practice_postal['K82019']
```

    A81001 TS18 1HU
    A81002 TS18 2AW
    A81003 TS25 1QU
    A81004 TS1 3BE
    A81005 TS14 7DJ
    A81006 TS18 2AT





    'HP21 8TR'




```python
assert practice_postal['K82019'] == 'HP21 8TR'
```

Now we can join `practice_postal` to `scripts`.


```python
joined = scripts[:]
for script in joined:
    script['post_code'] = practice_postal[script['practice']]
```


```python
joined[:2]
```




    [{'bnf_code': '0101010G0AAABAB',
      'items': 2,
      'practice': 'N81013',
      'bnf_name': 'Co-Magaldrox_Susp 195mg/220mg/5ml S/F',
      'nic': 5.98,
      'act_cost': 5.56,
      'quantity': 1000,
      'post_code': 'SK11 6JL'},
     {'bnf_code': '0101021B0AAAHAH',
      'items': 1,
      'practice': 'N81013',
      'bnf_name': 'Alginate_Raft-Forming Oral Susp S/F',
      'nic': 1.95,
      'act_cost': 1.82,
      'quantity': 500,
      'post_code': 'SK11 6JL'}]



Finally we'll group the prescription dictionaries in `joined` by `'post_code'` and sum up the items prescribed in each group, as we did in the previous question.


```python
group_by_postal = group_by_field(joined, ('post_code',))
```


```python
items_by_post = {}

for postal_code in group_by_postal:
    items_by_post[postal_code] = 0
    for script in group_by_postal[postal_code]:
        items_by_post[postal_code] += script['items']
        
# for script in joined:
#     if script['post_code'] in items_by_post:
#         items_by_post[script['post_code']] += script['items']
#     else:
#         items_by_post[script['post_code']] = script['items']
```


```python
count = 0
for i in items_by_post:
    print(i, items_by_post[i])
    if count == 5:
        break
    count += 1
```

    SK11 6JL 110071
    CW5 5NX 38797
    CW1 3AW 64104
    CW7 1AT 43164
    CH65 6TG 25090
    CH1 4DS 34915



```python
print(len(postal_totals))
```

    100



```python
postal_totals = sorted([(item, items_by_post[item]) for item in items_by_post])
print(len(postal_totals))
postal_totals = sorted([(item, items_by_post[item]) for item in items_by_post])[:100]
print(len(postal_totals))
```

    118
    100



```python
grader.score.pw__postal_totals(postal_totals)
```

    ==================
    Your score:  1.0
    ==================


## Question 4: items_by_region

Now we'll combine the techniques we've developed to answer a more complex question. Find the most commonly dispensed item in each postal code, representing the results as a list of tuples (`post_code`, `bnf_name`, amount dispensed as proportion of total). Sort your results ascending alphabetically by post code and take only results from the first 100 post codes.

**NOTE:** We'll continue to use the `joined` variable we created before, where we've chosen the alphabetically first postal code for each practice. Additionally, some postal codes will have multiple `'bnf_name'` with the same number of items prescribed for the maximum. In this case, we'll take the alphabetically first `'bnf_name'`.

Now we need to calculate the total items of each `'bnf_name'` prescribed in each `'post_code'`. Use the techniques we developed in the previous questions to calculate these totals. You should have 141196 `('post_code', 'bnf_name')` groups.


```python
total_items_by_bnf_post = {}
for script in joined:
    if (script['post_code'], script['bnf_name']) in total_items_by_bnf_post:
        total_items_by_bnf_post[(script['post_code'], script['bnf_name'])] += script['items']
    else:
        total_items_by_bnf_post[(script['post_code'], script['bnf_name'])] = script['items']

# total_items_by_bnf_post = ...
assert len(total_items_by_bnf_post) == 141196
```


```python
count = 0
for i in total_items_by_bnf_post:
    print(i, total_items_by_bnf_post[i])
    if count == 5:
        break
    count += 1
```

    ('SK11 6JL', 'Co-Magaldrox_Susp 195mg/220mg/5ml S/F') 5
    ('SK11 6JL', 'Alginate_Raft-Forming Oral Susp S/F') 3
    ('SK11 6JL', 'Sod Algin/Pot Bicarb_Susp S/F') 94
    ('SK11 6JL', 'Sod Alginate/Pot Bicarb_Tab Chble 500mg') 9
    ('SK11 6JL', 'Gaviscon Infant_Sach 2g (Dual Pack) S/F') 41
    ('SK11 6JL', 'Gaviscon Advance_Liq (Aniseed) (Reckitt)') 98



```python
# total_items_by_bnf_post = group_by_field(joined, ('post_code', 'bnf_name'))
# len(total_items_by_bnf_post)
total_items_by_bnf_post[('NE24 1DX', 'Adalat Ret_Tab 20mg')]
```




    18




```python
# count = 0
# for i in total_items_by_bnf_post:
#     print(i, total_items_by_bnf_post[i])
#     if count == 5:
#         break
#     count += 1
# total_items_by_bnf_post.items()
my_dict = {'a': 1, 'b': 2, 'c': 3 }
print(my_dict.values())
print(my_dict.keys())
print(my_dict.items())
print(list(my_dict.items()))
```

    dict_values([1, 2, 3])
    dict_keys(['a', 'b', 'c'])
    dict_items([('a', 1), ('b', 2), ('c', 3)])
    [('a', 1), ('b', 2), ('c', 3)]


Let's use `total_by_item_post` to find the maximum item total for each postal code. To do this, we will want to regroup `total_by_item_post` by `'post_code'` only, not by `('post_code', 'bnf_name')`. First let's turn `total_by_item_post` into a list of dictionaries (similar to `scripts` or `practices`) and then group it by `'post_code'`. You should have 118 groups in `total_by_item_post` after grouping it by `'post_code'`.


```python
total_items = []
for (post_code, bnf_name), total in list(total_items_by_bnf_post.items()): 
    total_items.append({
        'post_code': post_code,
        'bnf_name': bnf_name,
        'total': total
    })

len(total_items)
```




    141196




```python
# total_items_by_post = joined[:]
# for script in joined:
#         total_items_by_post.append(script)

total_items_by_post = group_by_field(total_items, ('post_code',))

len(total_items_by_post)
```




    118




```python
# total_items_by_post = {}
# for script in joined:
#     if script['post_code'] in total_items_by_post:
#         total_items_by_post[script['post_code']] += script['items']
#     else:
#         total_items_by_post[script['post_code']] = script['items']

# total_items = ...
assert len(total_items_by_post) == 118
```


```python
count = 0
for i in total_items_by_post:
    print(i, total_items_by_post[i][:3])
    if count == 4:
        break
    count += 1
```

    SK11 6JL [{'post_code': 'SK11 6JL', 'bnf_name': 'Co-Magaldrox_Susp 195mg/220mg/5ml S/F', 'total': 5}, {'post_code': 'SK11 6JL', 'bnf_name': 'Alginate_Raft-Forming Oral Susp S/F', 'total': 3}, {'post_code': 'SK11 6JL', 'bnf_name': 'Sod Algin/Pot Bicarb_Susp S/F', 'total': 94}]
    CW5 5NX [{'post_code': 'CW5 5NX', 'bnf_name': 'Sod Algin/Pot Bicarb_Susp S/F', 'total': 21}, {'post_code': 'CW5 5NX', 'bnf_name': 'Sod Alginate/Pot Bicarb_Tab Chble 500mg', 'total': 4}, {'post_code': 'CW5 5NX', 'bnf_name': 'Gaviscon_Liq Orig Aniseed Relief', 'total': 3}]
    CW1 3AW [{'post_code': 'CW1 3AW', 'bnf_name': 'Maalox_Susp 195mg/220mg/5ml S/F', 'total': 2}, {'post_code': 'CW1 3AW', 'bnf_name': 'Sod Alginate/Pot Bicarb_Tab Chble 500mg', 'total': 2}, {'post_code': 'CW1 3AW', 'bnf_name': 'Gaviscon_Liq Orig Aniseed Relief', 'total': 2}]
    CW7 1AT [{'post_code': 'CW7 1AT', 'bnf_name': 'Sod Algin/Pot Bicarb_Susp S/F', 'total': 53}, {'post_code': 'CW7 1AT', 'bnf_name': 'Gaviscon Infant_Sach 2g (Dual Pack) S/F', 'total': 41}, {'post_code': 'CW7 1AT', 'bnf_name': 'Gaviscon Advance_Liq (Aniseed) (Reckitt)', 'total': 80}]
    CH65 6TG [{'post_code': 'CH65 6TG', 'bnf_name': 'Gaviscon Infant_Sach 2g (Dual Pack) S/F', 'total': 11}, {'post_code': 'CH65 6TG', 'bnf_name': 'Gaviscon Advance_Liq (Aniseed) (Reckitt)', 'total': 112}, {'post_code': 'CH65 6TG', 'bnf_name': 'Gaviscon Advance_Tab Chble Mint(Reckitt)', 'total': 5}]



```python
from operator import itemgetter
```


```python
get_total = itemgetter('total')
get_total(total_items_by_post['SK11 6JL'][2])
```




    94



Now we will aggregate the groups in `total_by_item_post` to create `max_item_by_post`. Some `'bnf_name'` have the same item total within a given postal code. Therefore, if more than one `'bnf_name'` has the maximum item total in a given postal code, we'll take the alphabetically first `'bnf_name'`. We can do this by [sorting](https://docs.python.org/2.7/howto/sorting.html) each group according to the item total and `'bnf_name'`.


```python
test_list = total_items_by_post['SK11 6JL']
sorted(test_list, key=itemgetter('total'), reverse=True)
```




    [{'post_code': 'SK11 6JL',
      'bnf_name': 'Omeprazole_Cap E/C 20mg',
      'total': 3219},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Aspirin Disper_Tab 75mg',
      'total': 1955},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Lansoprazole_Cap 30mg (E/C Gran)',
      'total': 1952},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Simvastatin_Tab 40mg', 'total': 1908},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Paracet_Tab 500mg', 'total': 1564},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ramipril_Cap 10mg', 'total': 1449},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Atorvastatin_Tab 20mg', 'total': 1349},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Amlodipine_Tab 5mg', 'total': 1301},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Metformin HCl_Tab 500mg',
      'total': 1186},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ventolin_Evohaler 100mcg (200 D)',
      'total': 1117},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Amlodipine_Tab 10mg', 'total': 1116},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Levothyrox Sod_Tab 25mcg',
      'total': 1071},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Citalopram Hydrob_Tab 20mg',
      'total': 1070},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Levothyrox Sod_Tab 100mcg',
      'total': 1027},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Atorvastatin_Tab 40mg', 'total': 1021},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Levothyrox Sod_Tab 50mcg',
      'total': 1008},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Bendroflumethiazide_Tab 2.5mg',
      'total': 933},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Zapain_Tab 30mg/500mg', 'total': 929},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Clopidogrel_Tab 75mg', 'total': 910},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Sertraline HCl_Tab 50mg',
      'total': 815},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Tamsulosin HCl_Cap 400mcg M/R',
      'total': 793},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Amoxicillin_Cap 500mg', 'total': 783},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Sertraline HCl_Tab 100mg',
      'total': 776},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Amitriptyline HCl_Tab 10mg',
      'total': 775},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Simvastatin_Tab 20mg', 'total': 760},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lisinopril_Tab 20mg', 'total': 757},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Furosemide_Tab 40mg', 'total': 756},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Lansoprazole_Cap 15mg (E/C Gran)',
      'total': 738},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Salbutamol_Inha 100mcg (200 D) CFF',
      'total': 699},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Alendronic Acid_Tab 70mg',
      'total': 677},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fluoxetine HCl_Cap 20mg',
      'total': 640},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Bisoprolol Fumar_Tab 2.5mg',
      'total': 634},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Prednisolone_Tab 5mg', 'total': 612},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Atorvastatin_Tab 80mg', 'total': 590},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Bisoprolol Fumar_Tab 1.25mg',
      'total': 582},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ramipril_Cap 2.5mg', 'total': 573},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Warfarin Sod_Tab 1mg', 'total': 569},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Losartan Pot_Tab 100mg', 'total': 566},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Tramadol HCl_Cap 50mg', 'total': 557},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ramipril_Cap 5mg', 'total': 548},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Folic Acid_Tab 5mg', 'total': 524},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Docusate Sod_Cap 100mg', 'total': 519},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Atenolol_Tab 50mg', 'total': 517},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Gabapentin_Cap 300mg', 'total': 511},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Warfarin Sod_Tab 3mg', 'total': 500},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ferr Fumar_Tab 210mg', 'total': 499},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Citalopram Hydrob_Tab 10mg',
      'total': 497},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Atorvastatin_Tab 10mg', 'total': 465},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Laxido_Oral Pdr Sach (Orange) S/F',
      'total': 463},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lisinopril_Tab 10mg', 'total': 447},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Mirtazapine_Tab 15mg', 'total': 443},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Furosemide_Tab 20mg', 'total': 432},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Pravastatin Sod_Tab 40mg',
      'total': 431},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Colecal & Calc_Tab Chble 400u/1.5g',
      'total': 431},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hydroxocobalamin_Inj 1mg/ml 1ml Amp',
      'total': 428},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Losartan Pot_Tab 50mg', 'total': 417},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Doxycycline Hyclate_Cap 100mg',
      'total': 412},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Vit B Co Strong_Tab', 'total': 392},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Fenbid_Gel 5%', 'total': 392},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Zopiclone_Tab 7.5mg', 'total': 382},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Doxazosin Mesil_Tab 4mg',
      'total': 381},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Mirtazapine_Tab 45mg', 'total': 374},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Gliclazide_Tab 80mg', 'total': 362},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ranitidine HCl_Tab 150mg',
      'total': 360},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Finasteride_Tab 5mg', 'total': 357},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Bisoprolol Fumar_Tab 5mg',
      'total': 353},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Propranolol HCl_Tab 40mg',
      'total': 350},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Thiamine HCl_Tab 100mg', 'total': 341},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Cetirizine HCl_Tab 10mg',
      'total': 339},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Aspirin_Tab 75mg', 'total': 336},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Allopurinol_Tab 100mg', 'total': 331},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Metformin HCl_Tab 500mg M/R',
      'total': 328},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Latanoprost_Eye Dps 50mcg/ml',
      'total': 322},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Co-Codamol_Tab 8mg/500mg',
      'total': 313},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sitagliptin_Tab 100mg', 'total': 310},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Diazepam_Tab 2mg', 'total': 308},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Atenolol_Tab 25mg', 'total': 302},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Amitriptyline HCl_Tab 25mg',
      'total': 302},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Propranolol HCl_Tab 10mg',
      'total': 301},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Mirtazapine_Tab 30mg', 'total': 294},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ramipril_Cap 1.25mg', 'total': 293},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Naproxen_Tab 500mg', 'total': 288},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Loratadine_Tab 10mg', 'total': 286},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Dressit Ster Dress Pack',
      'total': 286},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Donepezil HCl_Tab 10mg', 'total': 281},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Felodipine_Tab 10mg M/R',
      'total': 276},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Fluclox Sod_Cap 500mg', 'total': 272},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clarithromycin_Tab 500mg',
      'total': 268},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Codeine Phos_Tab 30mg', 'total': 264},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Adcal-D3_Tab Chble (Tutti-Frutti)',
      'total': 257},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Preservision_Lutein Cap',
      'total': 257},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Doxazosin Mesil_Tab 2mg',
      'total': 253},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Metformin HCl_Tab 1g M/R',
      'total': 253},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Felodipine_Tab 5mg M/R', 'total': 248},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clenil Modulite_Inha 100mcg (200D)',
      'total': 248},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Losartan Pot_Tab 25mg', 'total': 247},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'DuoResp Spiromax_Inh 160mcg/4.5mcg(120D)',
      'total': 243},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Bisacodyl_Tab E/C 5mg', 'total': 241},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Rivaroxaban_Tab 20mg', 'total': 239},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Naproxen_Tab 250mg', 'total': 237},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lisinopril_Tab 5mg', 'total': 236},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Methotrexate_Tab 2.5mg', 'total': 235},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Colecal & Calc_Tab Chble 400u/1.25g',
      'total': 234},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Lactulose_Soln 3.1g-3.7g/5ml',
      'total': 230},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Amitriptyline HCl_Tab 50mg',
      'total': 228},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Bumetanide_Tab 1mg', 'total': 225},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ferr Sulf_Tab 200mg', 'total': 225},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Bisoprolol Fumar_Tab 10mg',
      'total': 223},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sildenafil_Tab 100mg', 'total': 223},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Zerobase Crm (Appl)', 'total': 221},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Senna_Tab 7.5mg', 'total': 219},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Allopurinol_Tab 300mg', 'total': 219},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Aspen_Sorbaderm Barrier Crm 92g',
      'total': 216},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Symbicort_Turbohaler 200mcg/6mcg (120 D)',
      'total': 214},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Propranolol HCl_Cap 80mg M/R',
      'total': 213},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Tiotropium_Pdr For Inh Cap 18mcg',
      'total': 212},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Zopiclone_Tab 3.75mg', 'total': 211},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Citalopram Hydrob_Tab 40mg',
      'total': 211},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Spironol_Tab 25mg', 'total': 208},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Monomil XL_Tab 60mg', 'total': 208},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Indapamide_Tab 2.5mg', 'total': 201},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Vensir XL_Cap 150mg', 'total': 201},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Gabapentin_Cap 100mg', 'total': 200},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Quetiapine_Tab 25mg', 'total': 198},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Fultium-D3_Cap 800u', 'total': 195},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Mebeverine HCl_Cap 200mg M/R',
      'total': 194},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Lercanidipine HCl_Tab 10mg',
      'total': 188},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'GlucoRX FinePoint Needles Pen Inj Screw',
      'total': 186},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Candesartan Cilexetil_Tab 8mg',
      'total': 184},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Co-Codamol_Tab 30mg/500mg',
      'total': 184},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Polyfield Nitrile Patient Pack Ster Dres',
      'total': 179},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Diazepam_Tab 5mg', 'total': 176},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Indapamide_Tab 1.5mg M/R',
      'total': 174},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Cerelle_Tab 75mcg', 'total': 172},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Apixaban_Tab 5mg', 'total': 171},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Inadine 5cm x 5cm Povidone Iodine Fabric',
      'total': 171},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'CliniFast 10.75cm x 5m (Yellow) Stkntte',
      'total': 170},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Perindopril Erbumine_Tab 8mg',
      'total': 168},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Hux D3_Cap 20 000u', 'total': 166},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Mometasone Fur_Aq N/Spy 50mcg (140 D)',
      'total': 165},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Vensir XL_Cap 75mg', 'total': 164},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Felodipine_Tab 2.5mg M/R',
      'total': 163},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Phenoxymethylpenicillin Pot_Tab 250mg',
      'total': 163},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Quinine Sulf_Tab 200mg', 'total': 163},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ins Lantus SoloStar_100u/ml 3ml Pf Pen',
      'total': 162},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sildenafil_Tab 50mg', 'total': 160},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'K-Lite 10cm x 4.5m M/Layer Compress Band',
      'total': 160},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hydroxychlor Sulf_Tab 200mg',
      'total': 158},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hyoscine Butylbrom_Tab 10mg',
      'total': 153},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ispag Husk_Gran Eff Sach 3.5g G/F S/F',
      'total': 153},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hydrocort/Miconazole Nit_Crm 1%/2%',
      'total': 149},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clinitas Carbomer Eye Gel',
      'total': 149},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Hydrocort_Crm 1%', 'total': 148},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'K-Soft 10cm x 3.5m M/Layer Compress Band',
      'total': 148},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Omeprazole_Cap E/C 10mg',
      'total': 147},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Dapagliflozin_Tab 10mg', 'total': 145},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Codeine Phos_Tab 15mg', 'total': 144},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Beconase_Aq Nsl Spy 50mcg (200 D) 22g',
      'total': 143},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Digoxin_Tab 62.5mcg', 'total': 142},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Quinine Sulf_Tab 300mg', 'total': 142},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Olanzapine_Tab 10mg', 'total': 141},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Trimethoprim_Tab 200mg', 'total': 140},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Loperamide HCl_Cap 2mg', 'total': 137},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Carbocisteine_Cap 375mg',
      'total': 137},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Lercanidipine HCl_Tab 20mg',
      'total': 136},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hypromellose_Eye Dps 0.5%',
      'total': 136},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Prochlpzine Mal_Tab 5mg',
      'total': 135},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Aspirin_Tab E/C 75mg', 'total': 134},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ezetimibe_Tab 10mg', 'total': 133},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Montelukast_Tab 10mg', 'total': 133},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Pravastatin Sod_Tab 20mg',
      'total': 132},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Digoxin_Tab 125mcg', 'total': 131},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Candesartan Cilexetil_Tab 16mg',
      'total': 127},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Simvastatin_Tab 10mg', 'total': 126},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ensure Plus_Milkshake Style Liq(10 Flav)',
      'total': 126},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Warfarin Sod_Tab 5mg', 'total': 125},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Candesartan Cilexetil_Tab 4mg',
      'total': 124},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Calceos_Tab Chble 500mg/400u',
      'total': 124},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Nitrazepam_Tab 5mg', 'total': 123},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Cyclizine HCl_Tab 50mg', 'total': 122},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lyrica_Cap 150mg', 'total': 122},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clenil Modulite_Inha 200mcg (200D)',
      'total': 119},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Priadel_Tab 400mg', 'total': 119},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sumatriptan_Tab 50mg', 'total': 119},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Nitrofurantoin_Cap 100mg M/R',
      'total': 119},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fostair_Inh 100mcg/6mcg (120D) CFF',
      'total': 118},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Olanzapine_Tab 5mg', 'total': 118},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Morph Sulf_Oral Soln 10mg/5ml',
      'total': 118},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'CliniFast 7.5cm x 5m (Blue) Stkntte Elas',
      'total': 118},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Procyclidine HCl_Tab 5mg',
      'total': 117},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sharpsguard 1L', 'total': 114},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Otomize_Ear Spy 5ml', 'total': 113},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Duloxetine HCl_Cap G/R 60mg',
      'total': 112},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Amoxicillin_Oral Susp 250mg/5ml S/F',
      'total': 110},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Brinzolamide_Eye Dps 10mg/ml',
      'total': 110},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clinipore 2.5cm x 5m Surg Adh Tape Perm',
      'total': 109},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Accrete D3_Tab', 'total': 108},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Doublebase_Gel', 'total': 108},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lorazepam_Tab 1mg', 'total': 107},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Doxazosin Mesil_Tab 1mg',
      'total': 106},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Prednisolone_Tab 1mg', 'total': 106},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Linagliptin_Tab 5mg', 'total': 104},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Dermol_Crm', 'total': 102},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Quetiapine_Tab 100mg', 'total': 100},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Rigevidon_Tab', 'total': 100},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'GlucoRx Nexus (Reagent)_Strips',
      'total': 99},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Gaviscon Advance_Liq (Aniseed) (Reckitt)',
      'total': 98},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Perindopril Erbumine_Tab 4mg',
      'total': 98},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Paracet_Oral Susp 250mg/5ml S/F',
      'total': 97},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Solifenacin_Tab 5mg', 'total': 97},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Macrogol_Co Oral Pdr Sach S/F',
      'total': 96},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Fenbid Fte_Gel 10%', 'total': 96},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Spiriva_Pdr For Inh Cap 18mcg',
      'total': 95},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Sod Algin/Pot Bicarb_Susp S/F',
      'total': 94},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lyrica_Cap 75mg', 'total': 94},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'H & R_Proshield Foam & Spy Skin Cleanser',
      'total': 94},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Atenolol_Tab 100mg', 'total': 93},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Nicorandil_Tab 10mg', 'total': 92},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Paroxetine HCl_Tab 20mg', 'total': 92},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Bisoprolol Fumar_Tab 3.75mg',
      'total': 91},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Pivmecillinam HCl_Tab 200mg',
      'total': 91},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ins NovoRapid_Penfill 100u/ml 3ml Cart',
      'total': 91},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Aveeno_Crm', 'total': 91},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Co-Codamol_Tab 15mg/500mg',
      'total': 90},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ins NovoRapid_FlexPen 100u/ml 3ml Pf Pen',
      'total': 90},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lisinopril_Tab 2.5mg', 'total': 89},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Apixaban_Tab 2.5mg', 'total': 88},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'DuoResp Spiromax_Inh 320mcg/9mcg (60 D)',
      'total': 88},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Butec_Transdermal Patch 10mcg/hr',
      'total': 88},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Active (Reagent)_Strips', 'total': 88},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Performa (Reagent)_Strips',
      'total': 88},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fexofenadine HCl_Tab 180mg',
      'total': 87},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Priadel_Tab 200mg', 'total': 87},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Epilim Chrono 500_Tab 500mg C/R',
      'total': 87},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lyrica_Cap 300mg', 'total': 86},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Seretide 500_Accuhaler 500mcg/50mcg(60D)',
      'total': 85},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Diprobase_Crm', 'total': 85},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Mebeverine HCl_Tab 135mg',
      'total': 84},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Clonazepam_Tab 500mcg', 'total': 84},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Diclofenac Sod_Gel 1.16%',
      'total': 82},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Nortriptyline_Tab 10mg', 'total': 81},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Co-Dydramol_Tab 10mg/500mg',
      'total': 81},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lamotrigine_Tab 100mg', 'total': 81},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ibuprofen_Tab 400mg', 'total': 81},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Relvar Ellipta_Inha 92mcg/22mcg (30 D)',
      'total': 81},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Trazodone HCl_Cap 50mg', 'total': 80},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Anastrozole_Tab 1mg', 'total': 80},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Tamoxifen Cit_Tab 20mg', 'total': 80},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Avamys_Nsl Spy 27.5mcg (120D)',
      'total': 80},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Influenza_Vac Inact 0.5ml Pfs',
      'total': 80},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lamotrigine_Tab 50mg', 'total': 79},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Baclofen_Tab 10mg', 'total': 79},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Gliclazide_Tab 30mg M/R', 'total': 79},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Incruse Ellipta_Inh 55mcg (30D)',
      'total': 78},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Methadone HCl_Mix 1mg/1ml S/F',
      'total': 78},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Chloramphen_Eye Dps 0.5%',
      'total': 78},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ranitidine HCl_Tab 300mg',
      'total': 77},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Paracet_Cap 500mg', 'total': 77},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Levetiracetam_Tab 500mg', 'total': 77},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Dermol 500_Lot', 'total': 77},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Irripod Sod Chlor Top Irrig 20ml',
      'total': 77},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Lansoprazole_Orodisper Tab 30mg',
      'total': 76},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Omeprazole_Cap E/C 40mg', 'total': 76},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Quetiapine_Tab 300mg', 'total': 76},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Tetralysal 300_Cap', 'total': 76},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fluticasone Fur_Nsl Spy 27.5mcg (120D)',
      'total': 76},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Allevyn Gentle Border 10cm x 10cm Wound',
      'total': 75},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Trospium Chlor_Cap 60mg M/R',
      'total': 74},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lamictal_Tab 100mg', 'total': 73},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Desogestrel_Tab 75mcg', 'total': 73},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Alfuzosin HCl_Tab 10mg M/R',
      'total': 73},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Phyllocontin Continus_Tab 225mg',
      'total': 72},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Dihydrocodeine Tart_Tab 30mg',
      'total': 72},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Mirabegron_Tab 50mg M/R', 'total': 72},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Azathioprine_Tab 50mg', 'total': 72},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Aspen_Sorbaderm No-Sting Barrier Film 1m',
      'total': 72},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Eplerenone_Tab 25mg', 'total': 71},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Latanoprost/Timolol_Eye Dps 50mcg/5mg/ml',
      'total': 71},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Esomeprazole_Tab E/C 20mg',
      'total': 70},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Candesartan Cilexetil_Tab 32mg',
      'total': 70},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Zolpidem Tart_Tab 10mg', 'total': 70},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Venlafaxine_Tab 37.5mg', 'total': 70},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Estradiol_Pess 10mcg', 'total': 70},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'CliniSupplies_ProSys Night Bag Non-Ster',
      'total': 70},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Terbut Sulf_B/A Inha 500mcg (100 D)',
      'total': 69},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Risedronate Sod_Tab 35mg',
      'total': 69},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Memantine HCl_Tab 20mg', 'total': 68},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Fusidic Acid_Crm 2%', 'total': 68},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Epaderm Oint', 'total': 67},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Betahistine HCl_Tab 8mg', 'total': 66},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Omnitest 3 (Reagent)_Strips',
      'total': 66},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Gaviscon Advance_Liq (Peppermint) S/F',
      'total': 65},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Bezafibrate_Tab 400mg M/R',
      'total': 65},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Sirdupla_Inh 125mcg/25mcg (120D)',
      'total': 65},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Duloxetine HCl_Cap G/R 30mg',
      'total': 65},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Butec_Transdermal Patch 5mcg/hr',
      'total': 65},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'FastClix Lancets 0.3mm/30 Gauge',
      'total': 65},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Enalapril Mal_Tab 20mg', 'total': 64},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Adalat LA 60_Tab 60mg', 'total': 64},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Zapain_Cap 30mg/500mg', 'total': 64},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Tegaderm Foam Adh Oval 10cm x 11cm Wound',
      'total': 64},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Movicol Plain_Paed Pdr Sach 6.9g',
      'total': 63},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Bisoprolol Fumar_Tab 7.5mg',
      'total': 63},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Propranolol HCl_Cap 160mg M/R',
      'total': 63},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Risperidone_Tab 1mg', 'total': 63},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Risperidone_Tab 500mcg', 'total': 62},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sinemet-Plus_Tab 125mg', 'total': 62},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Viscotears_Liq Gel 2mg/g',
      'total': 62},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hylo-Forte Sod Hyaluronate Eye Dps 0.2%',
      'total': 62},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Aviva (Reagent)_Strips', 'total': 61},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Bimatoprost_Eye Dps 100mcg/ml',
      'total': 61},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Rosuvastatin Calc_Tab 10mg',
      'total': 60},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Epilim Chrono 300_Tab 300mg C/R',
      'total': 60},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Aerochamber Plus', 'total': 60},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Symbicort_Turbohaler 400mcg/12mcg (60 D)',
      'total': 59},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Zolpidem Tart_Tab 5mg', 'total': 59},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lyrica_Cap 50mg', 'total': 59},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Calcichew D3 Fte_Tab Chble',
      'total': 59},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ivabradine_Tab 5mg', 'total': 58},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Chlorphenamine Mal_Tab 4mg',
      'total': 58},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lamictal_Tab 50mg', 'total': 58},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fludrocort Acet_Tab 100mcg',
      'total': 58},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Clotrimazole_Crm 1%', 'total': 58},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Octasa_MR Tab E/C 800mg', 'total': 57},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Glyceryl Trinit_Sub P/Spy 400mcg (180D)',
      'total': 57},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Temazepam_Tab 10mg', 'total': 57},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ticagrelor_Tab 90mg', 'total': 56},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Sirdupla_Inh 250mcg/25mcg (120D)',
      'total': 56},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Metformin HCl_Tab 850mg', 'total': 56},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Letrozole_Tab 2.5mg', 'total': 56},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Bimatoprost/Timolol_Eye Dps 300mcg/5mg',
      'total': 56},
     {'post_code': 'SK11 6JL', 'bnf_name': 'E45_Crm', 'total': 56},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Betameth Val_Crm 0.1%', 'total': 56},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Perindopril Erbumine_Tab 2mg',
      'total': 55},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Hydrocort_Tab 10mg', 'total': 55},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hydrocort Acet/Fusidic Acid_Crm 1%/2%',
      'total': 55},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Allevyn Gentle Border 7.5cm x 7.5cm Woun',
      'total': 55},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Epaderm Crm', 'total': 55},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Olanzapine_Tab 2.5mg', 'total': 54},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Metronidazole_Tab 400mg', 'total': 54},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Prednisolone_Tab E/C 2.5mg',
      'total': 54},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Clobet But_Crm 0.05%', 'total': 54},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Clobet But_Oint 0.05%', 'total': 54},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ketoconazole_Shampoo 2%', 'total': 54},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Aerochamber Plus + Adult/Child/Infant Ma',
      'total': 54},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Esomeprazole_Tab E/C 40mg',
      'total': 53},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fexofenadine HCl_Tab 120mg',
      'total': 53},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Tegretol Prolonged Release_Tab 200mg M/R',
      'total': 53},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Gabapentin_Tab 600mg', 'total': 53},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Co-Amoxiclav_Tab 500mg/125mg',
      'total': 53},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Nitrofurantoin_Cap 50mg', 'total': 53},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Mobile (Reagent)_Strips', 'total': 53},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Depo-Provera_Inj 150mg/ml 1ml Pfs',
      'total': 53},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sod Bicarb_Cap 500mg', 'total': 53},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Pravastatin Sod_Tab 10mg',
      'total': 52},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Quetiapine_Tab 200mg', 'total': 52},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Colchicine_Tab 500mcg', 'total': 52},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lacri-Lube_Ophth Oint', 'total': 52},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Betameth Val_Oint 0.1%', 'total': 52},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Pneumococcal_Vac 0.5ml Vl (23 Valent)',
      'total': 52},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Adalat LA 30_Tab 30mg', 'total': 51},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Simvastatin_Tab 80mg', 'total': 51},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Trazodone HCl_Cap 100mg', 'total': 51},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Venlafaxine_Tab 75mg', 'total': 51},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Betahistine HCl_Tab 16mg',
      'total': 51},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Solifenacin_Tab 10mg', 'total': 51},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Epilim Chrono 200_Tab 200mg C/R',
      'total': 50},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Clobazam_Tab 10mg', 'total': 50},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ins NovoMix 30_FlexPen 100u/ml 3ml PfPen',
      'total': 50},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Sod Cromoglicate_Eye Dps Aq 2%',
      'total': 50},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Rivaroxaban_Tab 15mg', 'total': 50},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Monomax XL_Tab 60mg', 'total': 49},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'C D Medical_Peel Easy Adh Remover Spy 50',
      'total': 49},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ibuprofen_Gel 10%', 'total': 49},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'H & R_Proshield Plus Skin Prote 115g',
      'total': 49},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Dipyridamole_Cap 200mg M/R',
      'total': 48},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Escitalopram_Tab 10mg', 'total': 48},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lamotrigine_Tab 25mg', 'total': 48},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Phenytoin_Sod Cap 100mg', 'total': 48},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Cyanocobalamin_Tab 50mcg',
      'total': 48},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Forceval_Cap', 'total': 48},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Chloramphen_Eye Oint 1%', 'total': 48},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'White Soft Paraf/Liq Paraf_(S) 50%/50%',
      'total': 48},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Resource_ThickenUp Pdr Clr',
      'total': 48},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Doxazosin Mesil_Tab 4mg M/R',
      'total': 47},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Oxybutynin HCl_Tab 2.5mg',
      'total': 47},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Neditol XL_Cap 4mg', 'total': 47},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Xailin Night Paraf Eye Oint P/F 5g',
      'total': 47},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Scheriproct_Oint', 'total': 46},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Dosulepin HCl_Cap 25mg', 'total': 46},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Cefalexin_Cap 500mg', 'total': 46},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Trimethoprim_Tab 100mg', 'total': 46},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Betameth Val/Fusidic Acid_Crm 0.1%/2%',
      'total': 46},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Quinine Bisulf_Tab 300mg',
      'total': 46},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Salazopyrin-En_Tab 500mg',
      'total': 45},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Doxazosin Mesil_Tab 8mg M/R',
      'total': 45},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Levetiracetam_Tab 250mg', 'total': 45},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Vagifem_Vag Tab 10mcg', 'total': 45},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Thiamine HCl_Tab 50mg', 'total': 45},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Hydrocort_Oint 1%', 'total': 45},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Cetraben Crm 500g', 'total': 45},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'K-Plus 10cm x 8.7m M/Layer Compress Band',
      'total': 45},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Rosuvastatin Calc_Tab 5mg',
      'total': 44},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Salbutamol_Inha B/A 100mcg (200 D) CFF',
      'total': 44},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Depakote_Tab G/R 250mg', 'total': 44},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Depakote_Tab G/R 500mg', 'total': 44},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Dosulepin HCl_Tab 75mg', 'total': 44},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Metoclopramide HCl_Tab 10mg',
      'total': 44},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Azithromycin_Tab 250mg', 'total': 44},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ins Levemir_FlexPen 100u/ml 3ml Pf Pen',
      'total': 44},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Nitromin_P/Spy 400mcg (180 D)',
      'total': 43},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Nystan_Susp (Ready Mixed)',
      'total': 43},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Aciclovir_Tab 400mg', 'total': 43},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Carbimazole_Tab 5mg', 'total': 43},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Anoro Ellipta_Inha 55mcg/22mcg (30D)',
      'total': 43},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fortisip Bottle_Liq (8 Flav)',
      'total': 43},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'K-Soft Long 10cm x 4.5m M/Layer Compress',
      'total': 43},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Peppermint Oil_Cap E/C 0.2ml',
      'total': 42},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Rosuvastatin Calc_Tab 20mg',
      'total': 42},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Shortec_Cap 5mg', 'total': 42},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Dermol 200_Shower Emollient',
      'total': 42},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Hydromol Oint', 'total': 42},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Gaviscon Infant_Sach 2g (Dual Pack) S/F',
      'total': 41},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Emozul_Cap E/C 20mg', 'total': 41},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Amiodarone HCl_Tab 200mg',
      'total': 41},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Beclomet Diprop/Formoterol_Inh100/6(120D',
      'total': 41},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Zomorph_Cap 10mg', 'total': 41},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Zolmitriptan_Tab 2.5mg', 'total': 41},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Rasagiline Mesil_Tab 1mg',
      'total': 41},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Phenoxymethylpenicillin_Soln 125mg/5ml',
      'total': 41},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Oilatum Jnr_Bath Additive',
      'total': 41},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hyabak Sod Hyaluronate Eye Dps 10ml P/F',
      'total': 41},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Longtec_Tab 10mg M/R', 'total': 40},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ins Lantus_100u/ml 3ml Cart',
      'total': 40},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Gliclazide_Tab 40mg', 'total': 40},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Tadalafil_Tab 20mg', 'total': 40},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Alfacalcidol_Cap 250ng', 'total': 40},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Calcichew D3_Tab Chble', 'total': 40},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Domperidone_Tab 10mg', 'total': 39},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Paracet_Tab Solb 500mg', 'total': 39},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sod Valpr_Tab 500mg M/R', 'total': 39},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hydrocort/Clotrimazole_Crm 1%/1%',
      'total': 39},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Nicorandil_Tab 20mg', 'total': 38},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Venlafaxine_Tab 225mg M/R',
      'total': 38},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Oramorph_Oral Soln 10mg/5ml',
      'total': 38},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lyrica_Cap 100mg', 'total': 38},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Estriol_Crm 0.1% + Applic',
      'total': 38},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fesoterodine Fumar_Tab 8mg M/R',
      'total': 38},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Foodlink_Complete Pdr (4 Flav)',
      'total': 38},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Betameth Val_Scalp Applic 0.1%',
      'total': 38},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fybogel_Gran Sach 3.5g Orange G/F S/F',
      'total': 37},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Scheriproct_Suppos', 'total': 37},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Spironol_Tab 50mg', 'total': 37},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ipratrop Brom_Inha 20mcg (200 D) CFF',
      'total': 37},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hyoscine Hydrob_Tab 300mcg',
      'total': 37},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sod Valpr_Tab 300mg M/R', 'total': 37},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Erythromycin_Tab E/C 250mg',
      'total': 37},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'CliniSupplies_ProSys Leg Bag Ster 500ml',
      'total': 37},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Irbesartan_Tab 300mg', 'total': 36},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Tranexamic Acid_Tab 500mg',
      'total': 36},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Salmeterol_Inha 25mcg (120 D) CFF',
      'total': 36},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Seretide 125_Evohaler 125mcg/25mcg(120D)',
      'total': 36},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Seretide 250_Evohaler 250mcg/25mcg(120D)',
      'total': 36},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Amoxicillin_Oral Susp 125mg/5ml S/F',
      'total': 36},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lymecycline_Cap 408mg', 'total': 36},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hypromellose_Eye Dps 0.3%',
      'total': 36},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Miconazole Nit_Crm 2%', 'total': 36},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Urgotul 5cm x 5cm Wound Dress Soft Polym',
      'total': 36},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Orlistat_Cap 120mg', 'total': 35},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Longtec_Tab 20mg M/R', 'total': 35},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Clonidine HCl_Tab 25mcg', 'total': 35},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Oxybutynin HCl_Tab 5mg', 'total': 35},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Donepezil HCl_Tab 5mg', 'total': 35},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Zetuvit E 10cm x 10cm Pfa Cellulose Dres',
      'total': 35},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Octasa_MR Tab E/C 400mg', 'total': 34},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Spironol_Tab 100mg', 'total': 34},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Melatonin_Tab 2mg M/R', 'total': 34},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Marol_Tab 100mg M/R (Teva)',
      'total': 34},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Carbamazepine_Tab 100mg', 'total': 34},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Inadine 9.5cm x 9.5 cm Povidone Iodine F',
      'total': 34},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Bricanyl_Turbohaler 500mcg (100 D)',
      'total': 33},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fluticasone/Formoterol_Inh 250/10mcg120D',
      'total': 33},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Zomorph_Cap 30mg', 'total': 33},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Gatalin XL_Cap 24mg', 'total': 33},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sitagliptin_Tab 50mg', 'total': 33},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Estriol_Crm 0.01% + Applic',
      'total': 33},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ibuprofen_Gel 5%', 'total': 33},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Brimonidine Tart_Eye Dps 0.2%',
      'total': 33},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Dorzolamide/Timolol_Eye Dps 2%/0.5%',
      'total': 33},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Durafiber 10cm x 10cm Wound Dress Protea',
      'total': 33},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Pregabalin_Cap 75mg', 'total': 33},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Conotrane_Crm', 'total': 33},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Optive Fusion Sod Hyaluronate Eye Dps 10',
      'total': 33},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Pantoprazole_Tab E/C 40mg',
      'total': 32},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Verapamil HCl_Tab 120mg M/R',
      'total': 32},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clenil Modulite_Inha 50mcg (200D)',
      'total': 32},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hydroxyzine HCl_Tab 25mg',
      'total': 32},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Prochlpzine Mal_Tab Buccal 3mg',
      'total': 32},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lyrica_Cap 25mg', 'total': 32},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Phenoxymethylpenicillin_Soln 250mg/5ml',
      'total': 32},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Colecal & Calc_Tab Chble 200u/1.25g',
      'total': 32},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ganfort_Eye Dps 300mcg/5mg',
      'total': 32},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Beclomet Diprop_Nsl Spy 50mcg (200 D)',
      'total': 32},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Metronidazole_Gel 0.75%', 'total': 32},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Flixonase_Aq Nsl Spy 50mcg (150 D)',
      'total': 32},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'PVC Ring Pess 50-80mm 1.25cm Thick',
      'total': 32},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Creon 10000_Cap E/C', 'total': 31},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fenofibrate_Tab 160mg (Micronised)',
      'total': 31},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Quetiapine_Tab 150mg', 'total': 31},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Co-Codamol Eff_Tab 8mg/500mg',
      'total': 31},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fencino_Transdermal Patch 25mcg/hr',
      'total': 31},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ciprofloxacin_Tab 500mg', 'total': 31},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Terbinafine HCl_Tab 250mg',
      'total': 31},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Glimepiride_Tab 4mg', 'total': 31},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Capasal_Therapeutic Shampoo',
      'total': 31},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Zetuvit E 20cm x 40cm Pfa Cellulose Dres',
      'total': 31},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Class 1 Stnd C/Kt B/Knee Stkn Elas Hose',
      'total': 31},
     {'post_code': 'SK11 6JL',
      'bnf_name': '3m Health Care_Cavilon Durable Barrier C',
      'total': 31},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Salbutamol_Inh Soln 2.5mg/2.5ml Ud',
      'total': 31},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Mometasone Fur_Oint 0.1%',
      'total': 31},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Adalat LA 20_Tab 20mg', 'total': 30},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Tegretol Prolonged Release_Tab 400mg M/R',
      'total': 30},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Trimethoprim_Oral Susp 50mg/5ml S/F',
      'total': 30},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Prednisolone_Tab E/C 5mg',
      'total': 30},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Diclofenac Sod_Tab E/C 50mg',
      'total': 30},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Betnovate_Crm 0.1%', 'total': 30},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Eumovate_Crm 0.05%', 'total': 30},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Calcipotriol/Betameth_Oint 0.005%/0.05%',
      'total': 30},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Rusch Brillant AquaFlate All Slc Foy Mal',
      'total': 30},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Rabeprazole Sod_Tab E/C 20mg',
      'total': 30},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ins NovoRapid_Inj 100u/ml 10ml Vl',
      'total': 30},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Naseptin_Crm', 'total': 30},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Allevyn Gentle Border 12.5cm x 12.5cm Wo',
      'total': 30},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Tiotropium_Pdr For Inh Cap 18mcg + Dev',
      'total': 29},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Paroxetine HCl_Tab 30mg', 'total': 29},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Epilim 200_Tab E/C 200mg',
      'total': 29},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Memantine HCl_Tab 10mg', 'total': 29},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Pioglitazone HCl_Tab 30mg',
      'total': 29},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Gedarel_Tab 30/150mcg', 'total': 29},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Colecal_Cap 800u', 'total': 29},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Difflam_P/Spy 0.15% 30ml',
      'total': 29},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hydromol_Bath & Shower Emollient',
      'total': 29},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Trimovate_Crm', 'total': 29},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Benzoyl Per/Clindamycin Phos_Gel 5%/1%',
      'total': 29},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Typhim VI_Vac 0.5ml Pfs', 'total': 29},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sharpsafe 1L', 'total': 29},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Glandosane A/Spy 50ml (App)',
      'total': 29},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Co-Codamol_Cap 30mg/500mg',
      'total': 29},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Seebri_Breezhaler Inh Cap 55mcg + Dev',
      'total': 29},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Atrauman 7.5cm x 10cm Ktd Polyester Dres',
      'total': 29},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Flecainide Acet_Tab 50mg',
      'total': 28},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Tildiem LA 200_Cap 200mg',
      'total': 28},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Buspirone HCl_Tab 5mg', 'total': 28},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Olanzapine_Tab 7.5mg', 'total': 28},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Keppra_Tab 500mg', 'total': 28},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Microgynon 30_Tab', 'total': 28},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Adcal-D3_Tab Chble (Lem)',
      'total': 28},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Timolol_Gel Eye Dps 0.25%',
      'total': 28},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Aq_Crm', 'total': 28},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Timodine_Crm', 'total': 28},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Senna_Oral Soln 7.5mg/5ml S/F',
      'total': 28},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Irbesartan_Tab 150mg', 'total': 28},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Pramipexole_Tab 88mcg', 'total': 28},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Mometasone Fur_Crm 0.1%', 'total': 28},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Zetuvit E 10cm x 20cm Pfa Cellulose Dres',
      'total': 28},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Aspen_Sorbaderm No-Sting Barrier Film Sp',
      'total': 28},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Sod Picosulf_Oral Soln 5mg/5ml S/F',
      'total': 27},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Carvedilol_Tab 25mg', 'total': 27},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Oxycodone HCl_Oral Soln 5mg/5ml S/F',
      'total': 27},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Matrifen_Patch 12mcg/hr', 'total': 27},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fencino_Transdermal Patch 12mcg/hr',
      'total': 27},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lamotrigine_Tab 200mg', 'total': 27},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lamictal_Tab 25mg', 'total': 27},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Cerazette_Tab 75mcg', 'total': 27},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Liq Paraf/Isopropyl Myrist_Gel 15%/15%',
      'total': 27},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Fucidin H_Crm', 'total': 27},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Systane Hydroxypropyl Guar Eye Dps 10ml',
      'total': 27},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Vit_Cap', 'total': 27},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Efudix_Crm 5%', 'total': 27},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Tegaderm Foam Adh Oval 6.9cm x 7.6cm Wou',
      'total': 27},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Creon 25000_Cap E/C', 'total': 26},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Olanzapine_Tab 15mg', 'total': 26},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sumatriptan_Tab 100mg', 'total': 26},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fluclox Sod_Oral Soln 125mg/5ml',
      'total': 26},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Clotrimazole_Crm 2%', 'total': 26},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clotrimazole_Pess 500mg/Crm 2%',
      'total': 26},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Cilest_Tab', 'total': 26},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Tolterodine_Tab 2mg', 'total': 26},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Piroxicam_Gel 0.5%', 'total': 26},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Oilatum_Crm (New Form) (Liq)',
      'total': 26},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Betnovate_Oint 0.1%', 'total': 26},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'GlucoRx Lancets 0.31mm/30 Gauge',
      'total': 26},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Unistik 3 Comfort Lancets 1.8mm/28 Gauge',
      'total': 26},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Trazodone HCl_Tab 150mg', 'total': 26},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Co-Codamol Eff_Tab 30mg/500mg',
      'total': 26},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lamictal_Tab 200mg', 'total': 26},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Pioglitazone HCl_Tab 45mg',
      'total': 26},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ranitidine HCl_Oral Soln 75mg/5ml S/F',
      'total': 25},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Emozul_Cap E/C 40mg', 'total': 25},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Pantoprazole_Tab E/C 20mg',
      'total': 25},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Movicol_Pdr Sach 13.8g (Lem & Lim)',
      'total': 25},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Valsartan_Cap 160mg', 'total': 25},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Risperidone_Tab 2mg', 'total': 25},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ins Humulin M3_KwikPen 100u/ml 3ml PfPen',
      'total': 25},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Kliovance_Tab', 'total': 25},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fusidic Acid_Viscous Eye Dps 1%',
      'total': 25},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Dorzolamide_Eye Dps 2%', 'total': 25},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Dermol 600_Bath Emollient',
      'total': 25},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Revaxis_Vac 0.5ml Pfs', 'total': 25},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'UCS Physical Debridement Pad',
      'total': 25},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Amisulpride_Tab 200mg', 'total': 24},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Nortriptyline_Tab 25mg', 'total': 24},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Topiramate_Tab 25mg', 'total': 24},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ins NovoMix 30_Penfill 100u/ml 3ml Cart',
      'total': 24},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Oxybutynin HCl_Tab 5mg M/R',
      'total': 24},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Glucose_Tab', 'total': 24},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Timolol_Eye Dps 0.25%', 'total': 24},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Diprobase_Oint', 'total': 24},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Betameth Val_Crm 0.025% (1 in 4)',
      'total': 24},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Enstilar_Foam Aero 50mcg/0.5mg/g',
      'total': 24},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Erythromycin/Zn Acet_Lot 40mg/Ml/12mg/ml',
      'total': 24},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Softclix Lancets 0.4mm/28 Gauge',
      'total': 24},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Vita-POS Paraf Eye Oint P/F',
      'total': 24},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Paracet_Oral Susp 250mg/5ml',
      'total': 24},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Pregabalin_Cap 150mg', 'total': 24},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Elleste Solo_Tab 1mg', 'total': 24},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Midazolam HCl_Inj 5mg/ml 2ml Amp',
      'total': 24},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Moxonidine_Tab 200mcg', 'total': 23},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Isosorbide Mononit_Tab 10mg',
      'total': 23},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Butec_Transdermal Patch 20mcg/hr',
      'total': 23},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Tegretol_Tab 200mg', 'total': 23},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clarithromycin_Oral Susp 125mg/5ml',
      'total': 23},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Contour Next (Reagent)_Strips',
      'total': 23},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Elleste Duet_Tab 1mg', 'total': 23},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Noriday_Tab 350mcg', 'total': 23},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Trospium Chlor_Tab 20mg', 'total': 23},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ensure Compact_Liq (4 Flav)',
      'total': 23},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fluticasone/Azelastine_Spy50/137mcg 120D',
      'total': 23},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Liq Paraf Light_Bath Add 63.4%',
      'total': 23},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Atrauman 5cm x 5cm Ktd Polyester Dress',
      'total': 23},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Mepore 7cm x 8cm Pfa + Adh Border Dress',
      'total': 23},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Optiflo S Cath Maint Soln 50ml',
      'total': 23},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ranolazine_Tab 375mg M/R',
      'total': 23},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Olanzapine_Tab 20mg', 'total': 23},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Shortec_Cap 10mg', 'total': 23},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'FreeStyle Optium (Reagent)_Strips',
      'total': 23},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Class 11 Stnd C/Kt B/Knee Stkn Elas Hose',
      'total': 23},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Lisinopril/Hydchloroth_Tab 20mg/12.5mg',
      'total': 22},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clomipramine HCl_Cap 50mg',
      'total': 22},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Escitalopram_Tab 20mg', 'total': 22},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Longtec_Tab 5mg M/R', 'total': 22},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Longtec_Tab 40mg M/R', 'total': 22},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Pizotifen Malate_Tab 500mcg',
      'total': 22},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Epilim 500_Tab E/C 500mg',
      'total': 22},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Topiramate_Tab 50mg', 'total': 22},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Topiramate_Tab 100mg', 'total': 22},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Nitrofurantoin_Tab 50mg', 'total': 22},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ethinylest/Levonorgest_Tab 30/150mcg',
      'total': 22},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Azathioprine_Tab 25mg', 'total': 22},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ensure Plus Juce_Liq Feed (6 Flav)',
      'total': 22},
     {'post_code': 'SK11 6JL', 'bnf_name': 'E45_Itch Relief Crm', 'total': 22},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clobetasol Prop_Oint 0.05%',
      'total': 22},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Eumovate_Oint 0.05%', 'total': 22},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Calcipotriol_Oint 50mcg/1g',
      'total': 22},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Mepore 10cm x 11cm Pfa + Adh Border Dres',
      'total': 22},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Optive 0.5% Carmellose Eye Dps 10ml',
      'total': 22},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Verapamil HCl_Tab 240mg M/R',
      'total': 22},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Morph Sulf_Cap 10mg M/R', 'total': 22},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Dutasteride_Cap 500mcg', 'total': 22},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Urgotul SSD 10cm x 12cm Wound Dress Soft',
      'total': 22},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clarithromycin_Oral Susp 250mg/5ml',
      'total': 22},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Durafiber 5cm x 5cm Wound Dress Protease',
      'total': 22},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Lansoprazole_Orodisper Tab 15mg',
      'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Laxido_Paed Plain Oral Pdr Sach 6.9g',
      'total': 21},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Digoxin_Tab 250mcg', 'total': 21},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Enalapril Mal_Tab 10mg', 'total': 21},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Verapamil HCl_Tab 40mg', 'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Epipen_Auto-Inj 1/1000 1mg/ml 0.3ml',
      'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Liraglutide_Inj 6mg/ml 3ml PF Pen',
      'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Dioralyte_Pdr Sach (Blkcurrant)',
      'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ensure TwoCal_Liq (4 Flav)',
      'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Timoptol-LA_Ophth Gel-Forming Soln 0.25%',
      'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Nasofan_Aq Nsl Spy 50mcg (150 D)',
      'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Saline Steripoule_Inh Soln 0.9% 2.5ml Ud',
      'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Diprosalic_Oint 0.05%/3%',
      'total': 21},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Fucibet_Crm', 'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Adapalene/Benzoyl Per_Gel 0.1%/2.5%',
      'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Promethazine HCl_Tab 25mg',
      'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Venlafaxine_Tab 150mg M/R',
      'total': 21},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Aciclovir_Tab 200mg', 'total': 21},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Mirabegron_Tab 25mg M/R', 'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Aquacel Ag 5cm x 5cm Wound Dress Proteas',
      'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Spiriva_Pdr For Inh Cap 18mcg+HandiHaler',
      'total': 21},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Aripiprazole_Tab 5mg', 'total': 21},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ursodeoxycholic Acid_Cap 250mg',
      'total': 20},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Mirtazapine_Orodisper Tab 45mg',
      'total': 20},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Gabapentin_Cap 400mg', 'total': 20},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Calcichew_Tab Chble 1.25g (500mg)',
      'total': 20},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Tizanidine HCl_Tab 2mg', 'total': 20},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Oilatum Plus_Bath Additive',
      'total': 20},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Amorolfine HCl_Nail Lacquer Kit 5% 5ml',
      'total': 20},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Diamorph HCl_Inj 5mg Amp',
      'total': 20},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'FreeStyle Optium B-Ketone Reagent_Strips',
      'total': 20},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'BD Viva Needles Pen Inj Screw On 4mm/32',
      'total': 20},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Metoprolol Tart_Tab 50mg',
      'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Propranolol HCl_Tab 80mg',
      'total': 19},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sotalol HCl_Tab 40mg', 'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Seretide 250_Accuhaler 250mcg/50mcg(60D)',
      'total': 19},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Desloratadine_Tab 5mg', 'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Cetirizine HCl_Oral Soln 1mg/1ml S/F',
      'total': 19},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Cinnarizine_Tab 15mg', 'total': 19},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sinemet_Tab 12.5mg/50mg', 'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ins Levemir_Penfill 100u/ml 3ml Cart',
      'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fesoterodine Fumar_Tab 4mg M/R',
      'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Complan_Shake Pdr Sach 57g (Sbery)',
      'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Calc Carb_Tab Chble 1.25g (500mg) S/F',
      'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Calc Carb_Tab Chble 1.5g (600mg) S/F',
      'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Baclofen_Oral Soln 5mg/5ml S/F',
      'total': 19},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Permethrin_Crm 5%', 'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Engerix B_Vac 20mcg/ml 1ml Pfs',
      'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clinipore 2.5cm x 10m Surg Adh Tape Perm',
      'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Coloplast_S2 Urine Drain Bag + Nrv + Stn',
      'total': 19},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Buscopan_Tab 10mg', 'total': 19},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Nebivolol_Tab 5mg', 'total': 19},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Amisulpride_Tab 100mg', 'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Epilim_Liq 200mg/5ml S/F',
      'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ovestin_Crm 1mg + Applic',
      'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Complan_Shake Pdr Sach 57g (Banana)',
      'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Allevyn Adh 7.5cm x 7.5cm Wound Dress Po',
      'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Atrauman 10cm x 20cm Ktd Polyester Dress',
      'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Durafiber 2cm x 45cm Wound Dress Proteas',
      'total': 19},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Co-Amilofruse_Tab 5mg/40mg',
      'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Flecainide Acet_Tab 100mg',
      'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Valsartan_Cap 80mg', 'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Adizem-XL_Cap 120mg', 'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ventolin_Nebules Soln 2.5mg/2.5ml Ud',
      'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Symbicort_Turbohaler 100mcg/6mcg (120 D)',
      'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Diazepam_Tab 10mg', 'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Keppra_Tab 250mg', 'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Clonazepam_Tab 2mg', 'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sod Valpr_Tab E/C 200mg', 'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Amoxicillin_Oral Susp 250mg/5ml',
      'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Azithromycin_Oral Susp 200mg/5ml',
      'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Fluconazole_Cap 50mg', 'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Fluconazole_Cap 150mg', 'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Aciclovir_Tab 800mg', 'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ibandronic Acid_Tab 150mg',
      'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Darifenacin Hydrob_Tab 15mg M/R',
      'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Regurin XL_Cap 60mg', 'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sod Chlor_Tab 600mg M/R', 'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Nutramigen 1 LIPIL_Pdr', 'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fluticasone Prop_Nsl Spy 50mcg (150 D)',
      'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Chlorhex HCl/Neomycin Sulf_Crm 0.1/0.5%',
      'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Chlorhex Glucon_Mthwsh 0.2%',
      'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Diprosalic_Scalp Applic 0.05%/2%',
      'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'QV Crm', 'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'ZeroAQS Crm', 'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hylo-Tear Sod Hyaluronate Eye Dps 0.1% P',
      'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'C D Medical_Clinifilm Wipes',
      'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Candesartan Cilexetil_Tab 2mg',
      'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Bezafibrate_Tab 200mg', 'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Temazepam_Tab 20mg', 'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Co-Careldopa_Tab 25mg/100mg',
      'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Co-Careldopa_Tab 25mg/100mg M/R',
      'total': 18},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Adcal-D3_Capl 750mg/200u',
      'total': 18},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ramipril_Tab 2.5mg', 'total': 17},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Valsartan_Cap 40mg', 'total': 17},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Warfarin Sod_Tab 500mcg', 'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Aclidinium Brom_Pdr For Inh 375mcg (60D)',
      'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fluticasone/Salmeterol_Inh 125/25mcg120D',
      'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Flutiform_Inha 250/10mcg (120 D)',
      'total': 17},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Aripiprazole_Tab 10mg', 'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Valproic Acid_Tab G/R 500mg',
      'total': 17},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Matrifen_Patch 25mcg/hr', 'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fencino_Transdermal Patch 50mcg/hr',
      'total': 17},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Levetiracetam_Tab 1g', 'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'GlucoGel_Gel 40% (Original)',
      'total': 17},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Norethist_Tab 5mg', 'total': 17},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Norethist_Tab 350mcg', 'total': 17},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Pred Fte_Eye Dps 1%', 'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Travoprost/Timolol_Eye Dps 40mcg/5mg',
      'total': 17},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Aveeno_Bath Oil', 'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'BD Microfine+ Needles Pen Inj Screw On 8',
      'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Carmize 0.5% Carmellose Eye Dps 10ml',
      'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Qvar 100_Inha 100mcg (200 D)',
      'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Promazine HCl_Oral Soln 25mg/5ml',
      'total': 17},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Concerta_XL Tab 36mg', 'total': 17},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Primidone_Tab 250mg', 'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Nystatin_Oral Susp 100 000u/ml',
      'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Glycopyrronium Brom_Inj 0.2mg/ml 1ml Amp',
      'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Zetuvit E 20cm x 20cm Pfa Cellulose Dres',
      'total': 17},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Tacrolimus_Oint 0.1%', 'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Urgotul 10cm x 10cm Wound Dress Soft Pol',
      'total': 17},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Cetraben Oint 450g', 'total': 17},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Gaviscon Advance_Tab Chble Mint(Reckitt)',
      'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Loperamide HCl_Tab 2mg', 'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sulfasalazine_Tab 500mg', 'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Co-Amilofruse_Tab 2.5mg/20mg',
      'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Enalapril Mal_Tab 5mg', 'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Isotard 40 XL_Tab 40mg', 'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Adalat Ret_Tab 10mg', 'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Montelukast_Tab Chble 5mg S/F',
      'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Loratadine_Oral Soln 5mg/5ml',
      'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Carbocisteine_Oral Soln 250mg/5ml',
      'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Vensir XL_Cap 225mg', 'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lyrica_Cap 200mg', 'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Keppra_Tab 1g', 'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sod Valpr_Tab 200mg M/R', 'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Oxytetracycline_Tab 250mg',
      'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clarithromycin_Tab 250mg',
      'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Itraconazole_Cap 100mg', 'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Glimepiride_Tab 1mg', 'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Saxagliptin_Tab 5mg', 'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hydroxycarbamide_Cap 500mg',
      'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Carmellose Sod_Eye Dps 0.5% 0.4ml Ud P/F',
      'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Calcipotriol/Betameth_Gel 0.005%/0.05%',
      'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'ViATIM_Vac D/Chamber 160u/25mcg 1ml Pfs',
      'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'CliniFast 3.5cm x 1m (Red) Stkntte Elasc',
      'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hypafix 5cm x 5m Surg Adh Tape Perm Non-',
      'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Optiflo G Cath Maint Soln 50ml',
      'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'ReplensMD Vag Moist 35g Tube',
      'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'CliniMed_LBF Ster No Sting Barrier Film',
      'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sotalol HCl_Tab 80mg', 'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Memantine HCl_Oral Soln 10mg/ml S/F',
      'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Exemestane_Tab 25mg', 'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Water For Inj_10ml Amp', 'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Complan_Shake Pdr Sach 57g (Choc)',
      'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Hydrocort/Miconazole Nit_Oint 1%/2%',
      'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Aciclovir_Crm 5%', 'total': 16},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'ActiFormCool 5cm x 6.5cm Wound Dress H/G',
      'total': 16},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Amiloride HCl_Tab 5mg', 'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ventolin_Accuhaler 200mcg (60 D)',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Salamol E-Breathe_Inh 100mcg (200 D) CFF',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fluticasone/Salmeterol_Inh 250/50mcg 60D',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fluticasone/Salmeterol_Inh 250/25mcg120D',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Chlorphenamine Mal_Oral Soln 2mg/5ml',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clomipramine HCl_Cap 10mg',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Rizatriptan_Orodisper Tab 10mg S/F',
      'total': 15},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Zonisamide_Cap 100mg', 'total': 15},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Phenytoin_Sod Cap 50mg', 'total': 15},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Glimepiride_Tab 2mg', 'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'FreeStyle Lite (Reagent)_Strips',
      'total': 15},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Estradiol_Tab 2mg', 'total': 15},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Celecoxib_Cap 200mg', 'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Travoprost_Eye Dps 40mcg/ml',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Benzydamine HCl_Mthwsh 0.15% S/F',
      'total': 15},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Balneum Plus_Crm', 'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Sudocrem_Antis Healing Crm',
      'total': 15},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Daktacort_Crm 2%/1%', 'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Atrauman 20cm x 30cm Ktd Polyester Dress',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Allevyn Adh 10cm x 10cm Wound Dress Poly',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'BD Microfine+ Needles Pen Inj Screw On 5',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'BD Micro-Fine Ultra Needles Pen Inj Scre',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Esomeprazole_Cap E/C 20mg',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Omeprazole_Tab Disper 20mg (E/C Pellets)',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Lisinopril/Hydchloroth_Tab 10mg/12.5mg',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Tiotropium_Inha 2.5mcg (60D) CFF + Dev',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Uniphyllin Continus_Tab 200mg',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Cyclizine Lact_Inj 50mg/ml 1ml Amp',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ins Apidra SoloStar_100u/ml 3ml PF Pen',
      'total': 15},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sando-K_Tab Eff 470mg', 'total': 15},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Heparinoid_Crm 0.3%', 'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Havrix Monodose_Vac 1440u/ml 1ml Pfs',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Tegaderm Foam Adh Oval 14.3cm x 15.6cm W',
      'total': 15},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Peptac_Liq (Peppermint) S/F',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Isosorbide Mononit_Tab 20mg',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Budesonide_Pdr For Inh 200mcg (100 D)',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fluticasone/Formoterol_Inh 125/5mcg 120D',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Chlorpromazine HCl_Tab 25mg',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Chlorpromazine HCl_Tab 50mg',
      'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Risperidone_Tab 4mg', 'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Lofepramine HCl_Tab 70mg',
      'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Shortec_Cap 20mg', 'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Zomorph_Cap 60mg', 'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Marol_Tab 200mg M/R (Teva)',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Marol_Tab 150mg M/R (Teva)',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Zolmitriptan_Orodisper Tab 2.5mg S/F',
      'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Pregabalin_Cap 50mg', 'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Carbamazepine_Tab 200mg', 'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ropinirole HCl_Tab 2mg', 'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Cefalexin_Cap 250mg', 'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Dapagliflozin_Tab 5mg', 'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Forxiga_Tab 10mg', 'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Pioglitazone HCl_Tab 15mg',
      'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Estradiol_Val Tab 2mg', 'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Elleste Duet_Tab 2mg', 'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Testosterone_Gel 2% (10mg per actuation)',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Duloxetine HCl_Cap G/R 20mg',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Zoladex LA_Implant 10.8mg SafeSystem Pfs',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Voltarol Emulgel_Aq Gel 1.16%',
      'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ketoprofen_Gel 2.5%', 'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Dovobet_Oint 0.005%/0.05%',
      'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Emla_Crm 5%', 'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Losartan Pot_Tab 12.5mg', 'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Adizem-XL_Cap 240mg', 'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'OxyNorm_Oral Soln 5mg/5ml S/F',
      'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Pregabalin_Cap 100mg', 'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lyrica_Cap 225mg', 'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Desmopressin Acet_Tab 200mcg',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ensure_Liq Feed (3 Flav)',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Complan_Shake Pdr Sach 57g (Milk)',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Depo-Medrone_Inj 40mg/1ml Vl',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'DuoTrav_Eye Dps 40mcg/ml / 5mg/ml',
      'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ipratrop Brom_Aq Nsl Spy 21mcg (180D)CFF',
      'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Terbinafine HCl_Crm 1%', 'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Isosorbide Mononit_Tab 60mg M/R',
      'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Escitalopram_Tab 5mg', 'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Colecal_Tab 1 000u', 'total': 14},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Sanatogen Complete_A-Z Tab OAD',
      'total': 14},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Carvedilol_Tab 12.5mg', 'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ivabradine_Tab 7.5mg', 'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Valproic Acid_Tab G/R 250mg',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clomipramine HCl_Cap 25mg',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Levetiracetam_Oral Soln 500mg/5ml S/F',
      'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sitagliptin_Tab 25mg', 'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ketostix (Reagent)_Strips',
      'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Estradiol_Val Tab 1mg', 'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Lucette_Tab 0.03mg/3mg', 'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Tolterodine_Tab 1mg', 'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'MagnaPhate_Tab Chble 97.2mg',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Mefenamic Acid_Tab 500mg',
      'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Dexameth_Eye Dps 0.1%', 'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Olopatadine_Eye Dps 1mg/ml',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Celluvisc_Eye Dps 1% 0.4ml Ud',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Nasonex_Aq N/Spy 50mcg (140 D)',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Benzydamine HCl_Spy 0.15% 30ml S/F',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Difflam_Oral Rinse Soln 0.15%',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clobetasol Prop_Crm 0.05%',
      'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Hydrocort_Crm 0.5%', 'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Zetuvit E (Non-Ster) 20cm x 40cm Pfa Cel',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Non-Woven Fabric Swab 7.5cm x 7.5cm Ster',
      'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Flexitol Urea Heel Balm', 'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'ReplensMD Vag Moist 6D Pack',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Liquifilm 1.4% Polyvinyl Alcohol Eye Dps',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fybogel_Gran Sach 3.5g Plain G/F S/F',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Tildiem LA 300_Cap 300mg',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Naftidrofuryl Oxal_Cap 100mg',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Dabigatran Etexilate_Cap 150mg',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fluticasone/Salmeterol_Inh 50/25mcg 120D',
      'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Concerta_XL Tab 27mg', 'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Co-Careldopa_Tab 12.5mg/50mg',
      'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Bicalutamide_Tab 150mg', 'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Folic Acid_Tab 400mcg', 'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Complan_Shake Pdr Sach 57g (Vanilla)',
      'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Alphosyl_Shampoo 2 In 1', 'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Optive Plus 0.5% Carmellose Eye Dps',
      'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Pregabalin_Cap 300mg', 'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Tegretol_Tab 100mg', 'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Azopt_Eye Dps 10mg/ml', 'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Activon Tulle 5cm x 5cm Manuka Honey Ste',
      'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'DebriSoft 10cm x 10cm Physical Debrideme',
      'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Medihoney_Barrier Crm', 'total': 13},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Kerraped All Purpose Boot Sml/Med/Lge/Ex',
      'total': 13},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Pentasa SR_Tab 500mg', 'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Movicol Plain_Pdr Sach 13.7g',
      'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Anusol HC_Oint', 'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Eplerenone_Tab 50mg', 'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Co-Tenidone_Tab 50mg/12.5mg',
      'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fenofibrate_Cap 200mg (Micronised)',
      'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Fenofibrate_Cap 267mg (Micronised)',
      'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Budesonide/Formoterol Inh B/A 100/6(120D',
      'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'AirFluSal Forspiro_Inh 500/50mcg (60D)',
      'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ondansetron HCl_Tab 4mg', 'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Matrifen_Patch 50mcg/hr', 'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Fluclox Sod_Cap 250mg', 'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Glucose_Gel 40%', 'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ethinylest/Desogestrel_Tab 30mcg/150mcg',
      'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Oxybutynin HCl_Tab 10mg M/R',
      'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Tadalafil_Tab 5mg', 'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Sildenafil_Tab 25mg', 'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Juvela_G/F Mix Wte', 'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Alfacalcidol_Cap 1mcg', 'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Dorzolamide/Timolol_EyeDps2/0.5% 0.2mlUd',
      'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Monopost_Eye Dps 50mcg/ml 0.2mlUd P/F',
      'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Miconazole_Oromucosal Gel 20mg/g S/F',
      'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Benzoyl Per_Gel 5%', 'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Alissa Olive Oil Ear Dps Ear Wax Softeni',
      'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Dermatonics Once Heel Balm',
      'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Zerodouble Gel', 'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Adalat Ret_Tab 20mg', 'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Pulmicort_Turbohaler 200mcg (100 D)',
      'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Seretide 100_Accuhaler 100mcg/50mcg(60D)',
      'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Haloperidol_Cap 500mcg', 'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Morph Sulf_Tab 30mg M/R', 'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Phenobarb_Tab 30mg', 'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Toujeo_300u/ml 1.5ml Pf SoloStar Pen',
      'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Ferr Glucon_Tab 300mg', 'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Care Olive Oil Ear Dps Ear Wax Softening',
      'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Cathejell Lidocaine C Lubricant Gel 8.5g',
      'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Ultibro Breezhaler_Pdr Inh Cap + Dev',
      'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Venlafaxine_Tab 75mg M/R',
      'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Colecal & Calc_Tab Eff 400u/1.5g (Lem)',
      'total': 12},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'K-ThreeC 10cm x 3m M/Layer Compress Band',
      'total': 12},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Carvedilol_Tab 6.25mg', 'total': 11},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Nifedipine_Cap 5mg', 'total': 11},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Clenil Modulite_Inha 250mcg (200D)',
      'total': 11},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Flutiform_Inha 125/5mcg (120 D)',
      'total': 11},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Levocetirizine_Tab 5mg', 'total': 11},
     {'post_code': 'SK11 6JL',
      'bnf_name': 'Solpadol_Tab Eff 30mg/500mg',
      'total': 11},
     {'post_code': 'SK11 6JL', 'bnf_name': 'Rivastigmine_Cap 3mg', 'total': 11},
     ...]




```python
max_item_by_post = []
groups = list(total_items_by_post.values())

for group in groups:
    max_total = sorted(group, key=itemgetter('total'), reverse=True)[0]
    max_item_by_post.append(max_total)
```


```python
max_item_by_post[:3]
```




    [{'post_code': 'SK11 6JL',
      'bnf_name': 'Omeprazole_Cap E/C 20mg',
      'total': 3219},
     {'post_code': 'CW5 5NX',
      'bnf_name': 'Omeprazole_Cap E/C 20mg',
      'total': 1419},
     {'post_code': 'CW1 3AW',
      'bnf_name': 'Omeprazole_Cap E/C 20mg',
      'total': 2364}]




```python
max_item_by_post = [sorted(group, key=itemgetter('total'), reverse=True)[0] for group in groups]
```


```python
max_item_by_post[:3]
```




    [{'post_code': 'SK11 6JL',
      'bnf_name': 'Omeprazole_Cap E/C 20mg',
      'total': 3219},
     {'post_code': 'CW5 5NX',
      'bnf_name': 'Omeprazole_Cap E/C 20mg',
      'total': 1419},
     {'post_code': 'CW1 3AW',
      'bnf_name': 'Omeprazole_Cap E/C 20mg',
      'total': 2364}]




```python
len(max_item_by_post)
```




    118




```python
# max_item_by_post = {}

# for postal_code in total_items_by_post:
#     max_item = 0
#     last_bnf_name = ''
# #     print(total_items_by_post[postal_code])
#     for script in total_items_by_post[postal_code]:
#         last_bnf_name = script['bnf_name']
# #         print(script)
#         if script['items'] > max_item:
#             max_item = script['items']
#         elif max_item == script['items']:
#             if last_bnf_name < script['bnf_name']:
#                 last_bnf_name = script['bnf_name']
#                 max_item = script['items']
#     max_item_by_post[postal_code] = max_item
            
        
```


```python
# count = 0
# for i in max_item_by_post:
#     print(i)
#     if count == 4:
#         break
#     count += 1
```

In order to express the item totals as a proportion of the total amount of items prescribed across all `'bnf_name'` in a postal code, we'll need to use the total items prescribed that previously calculated as `items_by_post`. Calculate the proportions for the most common `'bnf_names'` for each postal code. Format your answer as a list of tuples: `[(post_code, bnf_name, total)]`


```python
print(len(items_by_post))
print(len(max_item_by_post))
```

    118
    118



```python
items_by_region = max_item_by_post[:100]
# for postal_code in max_item_by_post:
#     for script in total_items_by_post[postal_code]:
#         max_item = 0
#         bnf_name = 
#     items_by_region.append((postal_code, 'Salbutamol_Inha 100mcg (200 D) CFF', max_item_by_post[postal_code]/items_by_post[postal_code]))
#     items_by_region.append((postal_code, '', items_by_post[postal_code]/max_item_by_post[postal_code]))
# items_by_region = [('B11 4BW', 'Salbutamol_Inha 100mcg (200 D) CFF', 0.0341508247)] * 100
```


```python

items_by_region = []
for item in max_item_by_post:
#     print(item)
    post_code = item['post_code']
    bnf_name = item['bnf_name']
    total = item['total']
    proportion = total/items_by_post[post_code]
    items_by_region.append((post_code, bnf_name, proportion))
    
```


```python
items_by_region = sorted(items_by_region)[:100]
len(items_by_region)
```




    100




```python
grader.score.pw__items_by_region(items_by_region)
```

    ==================
    Your score:  1.0
    ==================


*Copyright &copy; 2017 The Data Incubator.  All rights reserved.*
