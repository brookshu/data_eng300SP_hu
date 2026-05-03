
SSH into EC2 Instance:
1. Navigate to where .pem file is located
2. Go to EC2 instance and start it
3. Ensure connection to Northwestern WiFi or GlobalProtect VPN
4. Once started, press connect and copy/paste the example at the bottom of the page into terminal 

### Box 1:
**Use**: Connect to AWS. I used the AWS CLI. To mirror what I did:
1. Install the AWS CLI
2. Set up a file .aws/credentials as such:
```
[549787090008_mse-tl-dataeng300-EMR]
aws_access_key_id=<some key>
aws_secret_access_key=<some secret>
aws_session_token=<some token>
```

The access key ID, secret access key, and token should be acquired from the [Northwestern AWS Start Page](https://nu-sso.awsapps.com/start/#/). Click on the available account name, and then press "Access keys".

3. in terminal: ```aws configure sso```. Fill out the prompts as follows:
```
SSO session name: 549787090008_mse-tl-dataeng300-EMR
SSO start URL: https://nu-sso.awsapps.com/start/#/
SSO region: us-east-2
```
4. A popup will open in your browser. Log in
5. Run the box on the Jupyter notebook, it should succeed.

**Expected output**: None (success)

### Box 2:
**Use:** Create client to AWS S3, test connection by listing buckets
**Expected Output:** Should print a list of buckets in S3

### Box 3:
**Use:** Check if zip file with data exists locally. If it doesn't, download it. Then check if it's extracted, and if it isn't, unzip it. Then, for each file, in the dataset, upload it to S3 if it hasn't been already.
**Expected Output:**
Depends on whether or not files are pre-existing. At the end, it should say:
```
movies.dat already exists in S3, skipping.
ratings.dat already exists in S3, skipping.
users.dat already exists in S3, skipping.
```
or 
```
Uploading movies.dat...
Uploaded movies.dat to S3!
Uploading ratings.dat...
Uploaded ratings.dat to S3!
Uploading users.dat...
Uploaded users.dat to S3!
```
These outputs indicate success.

### Box 4:
**Use:** Read the data from the files
**Expected Output:** None (success)

### Box 5:
**Use:** import necessary libraries, choose model, and load tokenizer/encoder.
**Expected Output:** 
```
[transformers] DistilBertModel LOAD REPORT from: distilbert-base-uncased
Key                     | Status     |  | 
------------------------+------------+--+-
vocab_transform.weight  | UNEXPECTED |  | 
vocab_layer_norm.bias   | UNEXPECTED |  | 
vocab_transform.bias    | UNEXPECTED |  | 
vocab_layer_norm.weight | UNEXPECTED |  | 
vocab_projector.bias    | UNEXPECTED |  | 

Notes:
- UNEXPECTED:	can be ignored when loading from different task/architecture; not ok if you expect identical arch.
```

### Box 6:
**Use:** Extract year from movie title, prepare movie data for text-based modeling
**Expected Output:** ```Movies selected: 887```

### Box 7:
**Use:** Define helper functions for BERT encoding, which convert text for each movie into embeddings in vector space
**Expected Output:** None (success)

### Box 8:
**Use:** Save embedding data to output a copy into S3
**Expected Output:** None (success)

### Box 9:
**Use:** Uploading embedding data to S3
**Expected Output:** None (success)

### Box 10:
**Use:** Create helper mappings to interact with FAISS
**Expected Output:** None (success)

### Box 11:
**Use:** Define function to build text for user
**Expected Output:** None (success)

### Box 12:
**Use:** Define function for recommending movies
**Expected Output:** None (success)

### Box 13:
**Use:** Acquire a top user by grouping by size of ratings per user ID, then getting the 95th quantile and choosing a random user. Get recommendations for them.
**Expected Output:** None (success)

### Box 14:
**Use:** Get recommendations for a cold user. This would be a user with no ratings, so essentially we're grouping movies by rating count and average rating, then getting the top five.
**Expected Output:** None (success)

### Box 15:
**Use:** Get ratings and user information for the top user. Then getting the last itneraction time for the top user and format the results for the top user and cold user.
**Expected Output:** None (success)

### Box 16:
**Use:** Get the movies that the top user "liked" (greater than or equal to a 4/5 rating), turn them into vectors.
**Expected Output:** None (success)

### Box 17:
**Use:** Print the cold user info (just to check).
**Expected Output:** 
User Type: Cold user
Last Interaction: None
Summary: No interaction history available. Recommendations use popular highly-rated pre-1980 movies.

### Box 18:
**Use:** Print the top user info (just to check).
**Expected Output:** Something like:
User Type: Top user
Last Interaction: 2003-02-15T22:38:13
Summary:
  user_id: 3519
  num_interactions: 666
  avg_rating: 3.045045045045045
  gender: F
  age: 25
  occupation: 14
  zip: 11215

### Box 19:
**Use:** Print the top user rating history.
**Expected Output:** Five rows of their rating history. The user_id should be equal to the one shown in the previous box. It should also show the rating, timestamp, title, and genres.

### Box 20:
**Use:** Print the recommended movies for cold user and top user.
**Expected Output:** 
Cold user recs:
Star Wars: Episode IV - A New Hope (1977)
Godfather, The (1972)
Star Wars: Episode V - The Empire Strikes Back (1980)
Casablanca (1942)
One Flew Over the Cuckoo's Nest (1975)

Top user recs:
All the King's Men (1949)
Long Goodbye, The (1973)
To Sir with Love (1967)
Candidate, The (1972)
Buddy Holly Story, The (1978)

### Box 21:
**Use:** Vectorize all the movies, not just pre-1980s ones
**Expected Output:** None (success).

### Box 22:
**Use:** Check if user ID 999999 is taken
**Expected Output:** Empty table with user_id, gender, age, occupation, and zip

### Box 23:
**Use:** Define a function to search movies by name, then search for movies to rate.
**Expected Output:** Searches in the format:
Search for: Shawshank Redemption
     movie_id                             title genres  year
315       318  Shawshank Redemption, The (1994)  Drama  1994

### Box 24:
**Use:** Create my ratings with user ID 999999
**Expected Output:** None (success)

### Box 25:
**Use:** Concatenate my ratings to the other ratings and create my recommendations
**Expected Output:** My recommendations, formatted as a list of dictionaries:
{'movie_id': 2628,
  'title': 'Star Wars: Episode I - The Phantom Menace (1999)',
  'genres': 'Action|Adventure|Fantasy|Sci-Fi',
  'year': 1999,
  'score': 0.9702330231666565},
  ....

### Box 26:
**Use:** Create my profile (a DataFrame of my ratings), then save it to S3
**Expected Output:** ```uploaded my user profile to S3```

### Box 27:
**Use:** Format my profile as ```my_results```
**Expected Output:** None (success)

### Box 28:
**Use:** Save ```my_results``` to S3
**Expected Output:** ```uploaded my recommendations to S3```