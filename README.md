README

Goal

The goal of this project is to conduct an analysis of the quality of Wikipedia articles concerning leaders (politicians) from various countries. By examining article coverage and quality, I aim to uncover potential biases in Wikipedia’s representation of politicians across different countries and regions.

Data & API

I used two datasets:

	•	politicians_by_country.AUG.2024.csv: A list of Wikipedia article URLs about politicians from a wide range of countries.
	•	population_by_country_AUG.2024.csv: Contains country and regional populations. It includes rows that provide cumulative regional population counts, distinguished by having ALL CAPS values in the ‘geography’ field (e.g., AFRICA, OCEANIA).

APIs used:

	1.	MediaWiki API:Info: Allows me to retrieve the latest revision ID for a given article title.
	2.	ORES (Objective Revision Evaluation Service): Provides a prediction of the article’s quality class based on the revision ID.

Code Implementation

The project involves processing data and making API calls to retrieve article quality assessments. I incorporated sample code from Dr. David McDonald, which is licensed under CC-BY. More details can be found here.

Additionally, I used ChatGPT to help generate some of the functions, specifically:

Function to Get Latest Revision ID

    def get_latest_revision_id(article_title):
    params = {
        "action": "query",
        "format": "json",
        "titles": article_title,
        "prop": "revisions",
        "rvprop": "ids",
        "rvslots": "*"
    }
    
    # Throttle the request to respect rate limits
    time.sleep(0.2)
    
    response = requests.get(WIKIPEDIA_API_ENDPOINT, headers={'User-Agent': REQUEST_HEADERS['User-Agent']}, params=params)
    data = response.json()
    
    pages = data.get('query', {}).get('pages', {})
    for page_id in pages:
        revisions = pages[page_id].get('revisions', [])
        if revisions:
            return revisions[0]['revid']
    return None

Function to Get Article Quality Prediction

    def get_article_quality(revision_id):
    url = ORES_API_ENDPOINT.format(model_name=ORES_MODEL_NAME)
    data = {
        "rev_id": revision_id,
        "features": False
    }
    
    # Throttle the request to respect rate limits
    time.sleep(0.2)
    
    response = requests.post(url, headers=REQUEST_HEADERS, data=json.dumps(data))
    
    if response.status_code == 200:
        result = response.json()
        scores = result.get('enwiki', {}).get('scores', {})
        revision_scores = scores.get(str(revision_id), {})
        prediction = revision_scores.get('prediction', None)
        return prediction
    else:
        print(f"Error {response.status_code}: Unable to get quality prediction for revision ID {revision_id}")
        return None
   
  Adding a Progress Bar and Processing Articles
  

    for index, row in tqdm(politicians_df.iterrows(), total=politicians_df.shape[0], desc='Processing articles'):
    article_title = row['article_title']
    
    # Skip if article_title is None
    if not article_title:
        error_articles.append(row['url'])
        error_count += 1
        continue
    
    try:
        # Get the latest revision ID
        revision_id = get_latest_revision_id(article_title)
        
        if revision_id:
            # Get the article quality prediction
            quality = get_article_quality(revision_id)
            
            # Update the DataFrame
            politicians_df.at[index, 'revision_id'] = revision_id
            politicians_df.at[index, 'article_quality'] = quality
        else:
            error_articles.append(row['url'])
            error_count += 1
    except Exception as e:
        print(f"Error processing article '{article_title}': {e}")
        error_articles.append(row['url'])
        error_count += 1
    
    # Update progress bar postfix with error count
    tqdm.set_postfix(tqdm(), errors=error_count)

Analysis

I ran all the names of leaders in a loop, calling the APIs to get predictions for each article. I then assessed the results to compute the following metrics:

	1.	Top 10 countries by coverage: The 10 countries with the highest total articles per capita (in descending order).
	2.	Bottom 10 countries by coverage: The 10 countries with the lowest total articles per capita (in ascending order).
	3.	Top 10 countries by high quality: The 10 countries with the highest high-quality articles per capita (in descending order).
	4.	Bottom 10 countries by high quality: The 10 countries with the lowest high-quality articles per capita (in ascending order).
	5.	Geographic regions by total coverage: A rank-ordered list of geographic regions (in descending order) by total articles per capita.
	6.	Geographic regions by high-quality coverage: A rank-ordered list of geographic regions (in descending order) by high-quality articles per capita.

Running the Code

To run the analysis:

	1.	Ensure all required datasets (politicians_by_country.AUG.2024.csv and population_by_country_AUG.2024.csv) are in the working directory.
	2.	Install necessary Python libraries: requests, pandas, tqdm.
	3.	Set up your environment variables for API access tokens if required.
	4.	Run all cells in the Jupyter notebook sequentially.

Research Implications

Reflections on Findings

Through this project, I learned about the significant disparities in Wikipedia article coverage and quality across different countries and regions. It was surprising to find that some smaller or less affluent countries had relatively high articles per capita, while some larger or wealthier nations did not. This suggests that factors other than economic status, such as community engagement and cultural emphasis on knowledge sharing, can influence Wikipedia content.

What biases did I expect to find in the data (before I started working with it), and why?

I expected to find biases favoring English-speaking and Western countries due to the predominance of English on Wikipedia and higher internet penetration rates in these regions. I also anticipated that countries with greater technological infrastructure and educational resources would have more and higher-quality articles.

What (potential) sources of bias did I discover in the course of my data processing and analysis?

During data processing, I noticed that some countries had missing or zero population data, which affected per capita calculations. Additionally, the categorization of politicians and the completeness of articles varied widely, potentially due to differences in local engagement with Wikipedia and accessibility issues.

What might my results suggest about (English) Wikipedia as a data source?

The results suggest that English Wikipedia is not entirely representative of the global community. It tends to reflect the contributions of users from specific regions, leading to overrepresentation of certain countries and underrepresentation of others. This highlights the need to consider potential biases when using Wikipedia as a data source for research.

Can I think of a realistic data science research situation where using these data might create biased or misleading results, due to the inherent gaps and limitations of the data?

Yes, if a researcher uses English Wikipedia data to train a natural language processing model for global applications, the model might perform poorly on content related to underrepresented regions. For example, a sentiment analysis model might not accurately capture the nuances of political discourse in countries with less representation on Wikipedia, leading to biased or misleading conclusions.

How might a researcher supplement or transform this dataset to potentially correct for the limitations/biases I observed?

A researcher could incorporate data from Wikipedia editions in other languages to capture a more diverse range of articles. Additionally, combining Wikipedia data with other sources, such as local news outlets, governmental records, and international databases, could help create a more balanced dataset that better represents different regions and perspectives.

License

This project includes code and data that are licensed under various open-source licenses. Please refer to the individual files and data sources for specific licensing information.

	•	Sample code from Dr. David McDonald is licensed under CC-BY 4.0.
	•	Data retrieved from Wikipedia and associated APIs are subject to their respective licenses.

Acknowledgments

	•	Dr. David McDonald for providing sample code and guidance.
	•	ChatGPT for assisting in generating some of the code functions used in this project.
	•	The Wikimedia Foundation for providing access to Wikipedia data and APIs.

   
