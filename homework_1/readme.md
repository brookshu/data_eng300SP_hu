# Homework 1 README

### 1. Investigate missing data
This section starts by loading the dataset and getting oriented.
Box 0–1: Download the zipped CSV and read it into inventory with pandas.
Box 3: Print the column names to see the schema.
Box 4: Count missing values in the main columns we care about:
CARRIER CARRIER_NAME MANUFACTURE_YEAR NUMBER_OF_SEATS CAPACITY_IN_POUNDS AIRLINE_ID
This is the first missing-data audit
Box 5: I inspect a few rows of those columns to get a feel for what the missingness looks like.

Fixing North American Airlines
Box 7: Look for rows where both CARRIER_NAME and UNIQUE_CARRIER_NAME are "North American Airlines" and compare them against rows where CARRIER is present. Basically trying to infer what the missing carrier code should be. Seems to be NA.
Box 8–9: Separately inspect rows where CARRIER_NAME is missing and rows where CARRIER is missing.
Markdown box 10 Logic explanation: some values are technically missing in the file, but recoverable from context or external knowledge. For North American Airlines, the carrier code can be filled in. For rows missing a carrier name but having a code, the name can be inferred.
Box 11: Define an imputation helper, fill missing CARRIER values for North American Airlines with 'NA'
Box 12: Gather rows where either CARRIER or CARRIER_NAME is missing, then inspect distinct carrier codes among them.
Box 13–15: Codes like L4 and OH identified to appear elsewhere with non-missing names, then use those to fill missing CARRIER_NAME and UNIQUE_CARRIER_NAME.
Box 16: Check whether the carrier-name imputation worked.

Fixing missing manufacture year
Box 18: Inspecting rows with missing MANUFACTURE_YEAR.
Box 19: Defining a helper find(...) that filters aircraft by manufacturer, model, and optionally carrier name, sorting by year. This is a lookup tool to manually inspect similar aircraft. Worked out somewhat poorly because the tool needs exact matches
Box 20: Use the helper on Bombardier CL-600-2C10.
Box 21: Inspection of nearby serial numbers for the same carrier and model. key idea is that serial number seems to increase roughly with manufacture year.
Box 22: built an imputation function for MANUFACTURE_YEAR: convert SERIAL_NUMBER to numeric, then for each row missing year, find rows with the same MODEL and MANUFACTURER and restrict to serial numbers within ±100 of the goal row. Then fill with the median MANUFACTURE_YEAR in that local ballpark
Box 23: Print any rows still missing MANUFACTURE_YEAR after the imputation. Just to check

Missingness type for seats and capacity
Markdown box 24: question
Box 25: Inspection of rows missing CAPACITY_IN_POUNDS.
Box 26–28: Test whether missing capacity seems related to other variables: first by carrier name, then by model, then by manufacturer. This is exploratory evidence for whether the missingness depends on observed data. 
Box 29: Created binary indicator columns: MISSING_CAPACITY_IN_POUNDS, MISSING_NUMBER_OF_SEATS, then group by AIRCRAFT_TYPE and compute average missingness rates by type.
Box 30–31: printed grouped summaries showing which aircraft types have the highest missingness rates for seats and capacity.
Box 32: Plot the top missingness rates by aircraft type as a bar chart.
Markdown box 33 concluded that that the missingness looks like MAR, since it seems related to aircraft type and the other observed variables rather than being purely random. Suggested imputing with similar-aircraft averages or external lookup.

### 2. Inspect and standardize MANUFACTURER, MODEL, AIRCRAFT_STATUS, OPERATING_STATUS
This section is about cleaning string fields and using the cleaned values to improve imputation.
Box 35: Inspected specific suspicious AIRCRAFT_TYPE values like 673X, 159, 6586, etc. and print their corresponding model/manufacturer combinations. This is a manual sanity check for miscoded aircraft-type values.
Box 36: Looked at Embraer ERJ variants and their capacities to see whether some aircraft types are really formatting or typo issues rather than truly distinct planes.
Markdown box 37 Inferred that that some aircraft-type values may be missing a leading digit or otherwise botched, maybe by human error
Box 38: For every (MODEL, MANUFACTURER, AIRCRAFT_TYPE) combination with missing capacity, check whether matching rows exist. This is effectively testing whether same-type rows already contain usable capacity values. Showed that capacity had matching rows.
Box 39: Do the same thing for missing number of seats - didn't work as well
Markdown box 40: Conclusion: missing capacity can likely be imputed from rows with the same manufacturer/model/type, but missing seats are harder because the categorical fields themselves are inconsistent and need standardization first
Box 41: Impute CAPACITY_IN_POUNDS using the mode within each (MODEL, MANUFACTURER) group.
Box 42–45: Inspect and fill missing AIRLINE_ID values for carriers OH and L4 using IDs found in other rows.

Standardization pass
Markdown box 46: Introdce the inspection of the four categorical columns.
Box 47 Print unique values for: MANUFACTURER, MODEL, AIRCRAFT_STATUS, OPERATING_STATUS. basically is a raw-data audit for categorical noise.
Markdown box 48 Summarizing what I saw:
MANUFACTURER: whitespace/inconsistency
AIRCRAFT_STATUS: casing issues
OPERATING_STATUS: casing plus blanks
MODEL: needs more detailed inspection, too many values
Box 49: Looking at how many unique model values there are. Takeaway: the model field is messy and high-cardinality.
Box 50–52 Inspecting subsets of values containing keywords like "boeing", "crj", and "airbus" to identify inconsistencies, like case differences, hyphen/no hyphen, extra manufacturer text like “industries”, and spelling variations/typos
Box 53, defining and apply two important standardizers:
standardize_manufacturer: maps many messy variants to canonical names such as airbus, boeing, bombardier, etc.
standardize_model: lowercases, removes spaces and hyphens, and strips labels like passanger, passenger, psgr
This is the main cleanup box for string normalization.
Box 54: Inspection of the results by printing manufacturer values, value counts, and the number of distinct models after cleaning. Looked pretty good!
Box 55: Standardizing AIRCRAFT_STATUS by lowercasing and stripping whitespace.
Box 56: Standardizing OPERATING_STATUS by lowercasing, stripping whitespace, and converting anything other than y or n to n.
Box 57: Now that MODEL and MANUFACTURER are cleaner, I imputed NUMBER_OF_SEATS using the mode within (MODEL, MANUFACTURER).
### 3. Remove rows that still have missing values
This section turns the partially cleaned dataset into a fully complete dataset for modeling.
Box 59: Printed the row count before dropping missing values and also print the number of missing values per column.
Box 60 Create cleaned_inventory by dropping all rows with any remaining missing values, removing rows where NUMBER_OF_SEATS == 0 (realized I had to do this for boxcos), removing rows where CAPACITY_IN_POUNDS == 0. So this is the final subset for modeling.
Markdown box 61: Talkin about that this removes a huge portion of the data.  Note that AIRCRAFT_TYPE is especially difficult to impute, and also point out that some aircraft-type labels are ambiguous enough that standardizing them incorrectly could be risky.
### 4. Transformation and derivative variables
This section looks at distributions and applies Box-Cox transforms.
Markdown box 62: Introducing the goal: inspect skewness and histograms for NUMBER_OF_SEATS and CAPACITY_IN_POUNDS.
Box 63 Import plotting tools, compute skewness for both variables, plot side-by-side histograms with KDE curves. This is the raw-distribution analysis.
Box 64: printing some stats: mode of NUMBER_OF_SEATS and max of CAPACITY_IN_POUNDS, just to help interpret the range and concentration of values.
Box 65: applying Box-Cox transformations to both columns and store them as: NUMBER_OF_SEATS_boxcox, CAPACITY_IN_POUNDS_boxcox. This is intended to make data more normal.
Box 66: Ploting boxcox histograms.
Markdown box 67: Talking about the transformed plots and note that visually the shapes do not seem radically different, though the scales have changed a lot.
5. Feature Engineering
This section creates a new size category from seats.
Box 69: Create a new categorical variable SIZE by quartiles of NUMBER_OF_SEATS: SMALL, MEDIUM, LARGE, XLARGE
Box 70: Count plot of SIZE against OPERATING_STATUS. This checks whether operating status varies with aircraft size.
Box 71: Count plot of SIZE against AIRCRAFT_STATUS. This checks whether aircraft status varies with aircraft size.
6. Modeling Refresher - simple prediction tasks
Box 73: Print all columns in cleaned_inventory to decide which variables to use.
Box 74: Define predictor sets: to predict NUMBER_OF_SEATS, use CAPACITY_IN_POUNDS, YEAR, AIRLINE_ID; to predict CAPACITY_IN_POUNDS, use NUMBER_OF_SEATS, YEAR, AIRLINE_ID
Box 75 import train_test_split, LinearRegression, RandomForestRegressor, and root_mean_squared_error; then define X and y for both tasks and split each into 80/20 train/test sets
Box 76: Train the linear regression model to predict NUMBER_OF_SEATS, generate train and test predictions, and compute RMSE on both.
Box 77: Do the same linear regression process for predicting CAPACITY_IN_POUNDS.
Box 78: Train a random forest regressor for predicting NUMBER_OF_SEATS, then compute train and test RMSE.
Box 79: Same random forest process for predicting CAPACITY_IN_POUNDS.
Markdown box 80: Results show the problem is highly predictable. Seats and capacity are strong predictors of each other, year may not add much, aircraft model numbers may contain useful extra signal, and random forest appears stronger overall, but with some overfitting