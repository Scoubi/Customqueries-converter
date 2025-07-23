# Customqueries-converter
This Repo contains steps to convert BloodHound Legacy Customqueries.json to the new format used by BloodHound Community Edition (BHCE)

## Acknowledgments
This method is inspired by [REDSEERcurity Spotlight](https://medium.com/seercurity-spotlight) Blog Post [Make Bloodhound Cool Again: Migrating Custom Queries from Legacy BloodHound to BloodHound CE](https://medium.com/seercurity-spotlight/make-bloodhound-cool-again-migrating-custom-queries-from-legacy-bloodhound-to-bloodhound-ce-83cffcfe5b64) 

I tried their method and got an error message, so I decided see if ChatGPT could help figure out the error. 

## Method 

1. You will need your `customqueries.json` file. It can be found in a specific folder on each OS
   - Windows : C:\Users\[USERNAME]\AppData\Roaming\BloodHound\customqueries.json
   - MacOS : /Users/[USERNAME]/Library/Application Support/bloodhound/customqueries.json
   - Linux : ~/.config/bloodhound/customqueries.json
  
Alternatively you can use one of the `customqueries.json` file that are available on GitHub. 

2. Convert your Legacy json into a format BHCE's API can consume
This query
- Ignore the queries that have selectors in them (they use the `"final": false` for the first query)
- Make sure the query as a name
- Write a file named `bhce_customqueries.json`

``` bash
jq '[.queries[] | . as $parent | .queryList[] | select(.final) | {name: (.title // $parent.name), query: .query, description: $parent.name}]' customqueries.json > bhce_customqueries.json
```

3. In your BHCE instance, grab the `JTW Bearer token`

4. Modify the following bash script
- Make sure you have the right URL in `API_URL`
- Replace `"eyJhb..."` with the actual JTW Token you found in the previous step
- Save the script to a file like `up_queries.sh`

``` bash
#!/bin/bash

# Path to valid CE formatted JSON file
INPUT_FILE="newformat_customqueries.json"

# API endpoint and token
#For quick tests or one-time calls, the JWT used by your browser may be the simplest route. 
#The API will accept calls using the following header structure in the HTTP request:
#'Authorization': Bearer $JWT_TOKEN
#If you open the Network tab within your browser, you will see calls against the API made utilizing this structure.

API_URL="http://localhost:8080/api/v2/saved-queries"
JWT_TOKEN="eyJhb..."

# Loop through each query and upload
jq -c '.[]' "$INPUT_FILE" | while read -r query; do
 curl -X POST \
 "$API_URL" \
 -H "Content-Type: application/json" \
 -H "Authorization: Bearer $JWT_TOKEN" \
 --data-raw "$query"
echo "" 
done
```

5. Upload the queries
- In your terminal run the following command : `bash up_queries.sh`

## Conclusion

This was tested using BloodHound Legacy Version: 4.3.1 and [BloodHound-CE](https://github.com/SpecterOps/BloodHound) v8.0.0 on MacOS
