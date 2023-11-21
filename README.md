# nosql challenge

# INTRODUCTION

The UK Food Standards Agency evaluates various establishments across the United Kingdom, and gives them a food hygiene rating. You've been contracted by the editors of a food magazine, Eat Safe, Love, to evaluate some of the ratings data in order to help their journalists and food critics decide where to focus future articles.

# Part 1: Database and Jupyter Notebook Set Up

1. Use NoSQL_setup_starter.ipynb, Imported the data provided in the establishments.json file from the Terminal. Named the database uk_food and the collection establishments. Copied the text used to import the data from the Terminal to a markdown cell in the notebook.

  ** mongoimport --type json -d uk_food -c establishments --drop --jsonArray establishments.json**

2. Within the notebook, import the libraries needed: PyMongo and Pretty Print (pprint).

3. Create an instance of the Mongo Client.

4. Confirm that the database has been created and loaded the data properly:
	a) List the databases in MongoDB. Confirm that uk_food is listed.
	b) List the collection(s) in the database to ensure that establishments is available.
	c) Find and display one document in the establishments collection using find_one and display with pprint.

5. Assign the establishments collection to a variable to prepare the collection for use.

# Part 2: Update the Database

Use NoSQL_setup_starter.ipynb for this section. The magazine editors have some requested modifications for the database before you can perform any queries or analysis for them. Make the following changes to the establishments collection:

1. An exciting new halal restaurant just opened in Greenwich, but hasn't been rated yet. The magazine has asked you to include it in your analysis. Add the following information to the database:

{
    "BusinessName":"Penang Flavours",
    "BusinessType":"Restaurant/Cafe/Canteen",
    "BusinessTypeID":"",
    "AddressLine1":"Penang Flavours",
    "AddressLine2":"146A Plumstead Rd",
    "AddressLine3":"London",
    "AddressLine4":"",
    "PostCode":"SE18 7DY",
    "Phone":"",
    "LocalAuthorityCode":"511",
    "LocalAuthorityName":"Greenwich",
    "LocalAuthorityWebSite":"http://www.royalgreenwich.gov.uk",
    "LocalAuthorityEmailAddress":"health@royalgreenwich.gov.uk",
    "scores":{
        "Hygiene":"",
        "Structural":"",
        "ConfidenceInManagement":""
    },
    "SchemeType":"FHRS",
    "geocode":{
        "longitude":"0.08384000",
        "latitude":"51.49014200"
    },
    "RightToReply":"",
    "Distance":4623.9723280747176,
    "NewRatingPending":True
}


2. Find the BusinessTypeID for "Restaurant/Cafe/Canteen" and return only the BusinessTypeID and BusinessType fields.

3. Update the new restaurant with the found BusinessTypeID.

4. The magazine is not interested in any establishments in Dover, so check how many documents contain the Dover Local Authority. Then, remove any establishments within the Dover Local Authority from the database, and check the number of documents to ensure they were deleted.

5. Some of the number values are stored as strings, when they should be stored as numbers.
	a) Use update_many to convert latitude and longitude to decimal numbers.
	b) Use update_many to convert RatingValue to integer numbers.

# Part 3: Exploratory Analysis

Eat Safe, Love has specific questions they want you to answer, which will help them find the locations they wish to visit and avoid.

Use NoSQL_analysis_starter.ipynb, RatingValue refers to the overall rating decided by the Food Authority and ranges from 1-5. The higher the value, the better the rating.

	a) This field also includes non-numeric values such as 'Pass', where 'Pass' means that the establishment passed their inspection but isn't given a number rating. We will coerce non-numeric values to 		    nulls during the database setup before converting ratings to integers.
	b) The scores for Hygiene, Structural, and ConfidenceInManagement work in reverse. This means, the higher the value, the worse the establishment is in these areas.

Use the following questions to explore the database, and find the answers, so you can provide them to the magazine editors.

	a) Use count_documents to display the number of documents contained in the result.

	b) Display the first document in the results using pprint.

	c) Convert the result to a Pandas DataFrame, print the number of rows in the DataFrame, and display the first 10 rows.

1. Which establishments have a hygiene score equal to 20?
	** Find the establishments with a hygiene score of 20
	query = {'scores.Hygiene': 20}

	** Use count_documents to display the number of documents in the result
	results = establishments.find(query)
	results_count = establishments.count_documents(query)

	** Display the first document in the results using pprint
	pprint(results[0])
	print("Number of documents in result:",results_count)


2. Which establishments in London have a RatingValue greater than or equal to 4?
	** Find the establishments with London as the Local Authority and has a RatingValue greater than or equal to 4.
	match_query = {'AddressLine1':{'$regex':"London"},'RatingValue': {'$gte': 4}}
	results = establishments.find(match_query)
	results_count = establishments.count_documents(match_query)

	** Display the first document in the results using pprint
	pprint(results[0])
	print("Number of documents in result:",results_count)
 
3. What are the top 5 establishments with a RatingValue of 5, sorted by lowest hygiene score, nearest to the new restaurant added, "Penang Flavours"?
	** Search within 0.01 degree on either side of the latitude and longitude.
	** Rating value must equal 5
	** Sort by hygiene score
	degree_search = 0.01
	latitude = 51.49014200
	longitude = 0.08384000

	latitude_range = {'$gte': latitude - degree_search, '$lte': latitude + degree_search}
	longitude_range = {'$gte': longitude - degree_search, '$lte': longitude + degree_search}

	query = {
    	'RatingValue': 5,
    	'geocode.latitude': latitude_range,
    	'geocode.longitude': longitude_range
	}
	sort =  [('scores.Hygiene', 1)]
	limit = 5

	** Print the results
	results = list(establishments.find(query).sort(sort).limit(limit))


4. How many establishments in each Local Authority area have a hygiene score of 0? Sort the results from highest to lowest, and print out the top ten local authority areas.
	** Create a pipeline that: 
	**  Matches establishments with a hygiene score of 0
	**  Groups the matches by Local Authority
	**  Sorts the matches from highest to lowest
	match_query = {'$match': {'scores.Hygiene': 0}}
	group_query = {'$group': {'_id': "$LocalAuthorityName", 'count': { '$sum': 1 }}}
	sort_values = {'$sort': { 'count': -1 }}
	pipeline = [match_query, group_query, sort_values]
	results = list(establishments.aggregate(pipeline))

	** Print the number of documents in the result
	print("Number of documments in result: ", len(results))

	** Print the first 10 results
	pprint(results[0:10])


The result for the above queries are stored in sepaerate dataframes result_df, result_df_1, result_df_2, result_df_3 respectively.
 
