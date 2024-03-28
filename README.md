# RocksetGPT
Build your own Personal Rockset Engineer. Create & Manage Rockset Collections, Workspaces, Query Lambdas, & more. Write Rockset SQL. Deploy Changes. Add, Update, Delete Documents - and more! Below is a step-by-step guide on how to get started! 

**Requirements:** 
- Rockset API Key
  - You can create a Rockset account for FREE and get $300 in credits by clicking [here](https://rockset.com/create/).
  - Create an API key in the [API Keys tab of the Rockset Console](https://console.rockset.com/apikeys). The region can be found in the dropdown menu at the top of the page. For more information, refer to [Rockset's API Reference](https://docs.rockset.com/documentation/reference/rest-api).
- ChatGPT Plus subscription
  - To create an account on OpenAI go [here](https://openai.com)
  
Check out these [Google Slides](https://docs.google.com/presentation/d/1TVqCDd8YQxQAxosvT6-ZCX4ERidTAAklQGRiBGnnCto/edit#slide=id.g2c731a7967e_0_295) for background on custom GPTs.

This project involves two steps: (1) Build a RocksetSpecGPT to create custom OpenAPI spec and (2) Build a RocksetGPT with those specs. Optionally, you can skip part 1 and use the specs included in this repository to build RocksetGPT. Just make sure to update `YOUR_REGION` with your region (`usw2a1`, `use1a1`, `euc1a1`, `euw1a1`, `apt2a1`).

## Part 1 - RocksetSpecGPT
<img width="423" alt="Screenshot 2024-03-28 at 12 06 48 AM" src="https://github.com/sofia099/RocksetGPT/assets/59860423/5f86b941-36ee-40a1-830b-5dc6393290c6">

To build RocksetGPT, we need an OpenAPI spec of all the endpoints we want our custom GPT to access. However, OpenAI has a few limitations on these specs including a limit on the number of actions (30) and the total size of the spec (1MB). There are some undisclosed limiations as well including enforcing server URLs to be a literal string, not allowing `$ref` in the `requestBody`s, and not including a `security` schema. I also discovered that `responses` are not necessary so that can be ommitted to ensure we stay below the 1MB limit. To handle this requirements, let's build a RocksetSpecGPT that will do this work for us:

1. Under "Explore GPTs" in the ChatGPT console, select "Create" in the upper right corner.
2. Give your GPT a name, description, and (optionally) a profile image.
3. Give your GPT a set of instructions (prompts):
    ```
    Your Knowledge includes an OpenAPI v3 spec of Rockset's REST API. The user will provide a list of operationIds it wants to include in the final spec and the region of their Rockset account. If they forget to provide either one, please ask them for it. Regions can be `usw2a1`, `use1a1`, `euc1a1`, `euw1a1`, or `apt2a1`. Once you have a list of operationIds and a region, conduct the following steps.
    
    Step 1: Extract the paths for all the operationIds but drop '/v1' from them. These will be the paths of the final spec.
    Step 2: Remove the responses. These are not necessary in the final spec.
    Step 3: Include info but do not include info: description in the final spec.
    Step 4: Add servers url "https://api.usw2a1.rockset.com/v1" but replace {region} in the url with the region the user provided.
    Step 5: The final spec can't have any $ref. Please iterative and replace $ref —no matter how deeply nested— with its definition until there are no more instances of $ref in the final spec. The definitions can be found in the original spec.
    Step 6: Verify that the spec OpenAPI 3.0 compatible and adjust as needed.
    Step 7: Provide a downloadable link to the user when done.
    ```
  
4. Give your GPT knowledge by uploading files to its Knowledge bank. For the RocksetSpecGPT, let's upload the full Rockset OpenAPI v3 spec which can be downloaded from the Rockset openapi public Github repository: https://github.com/rockset/openapi/tree/master
5. Enable the "Code Interpreter" built-in capability so that it can interpret the spec we just uploaded.
6. Test and you're done! You can test it's correct by sending the following request to your Chat bot:
   ```
   Create a spec for the following operations:
    query
    listWorkspaces
    getWorkspace
    createWorkspace
    deleteWorkspace
    workspaceCollections
    getCollection
    createCollection
    deleteCollection
    addDocuments
    patchDocuments
    deleteDocuments
    listQueryLambdasInWorkspace
    createQueryLambda
    updateQueryLambda
    deleteQueryLambda
    createQueryLambdaTag
    deleteQueryLambdaTag
    executeQueryLambdaByTag
    createScheduledLambda
    deleteScheduledLambda
    getScheduledLambda
    updateScheduledLambda
   ```
   The response should resemble the spec in this repository. Feel free to add more operations to this list and generate your own spec!
   
## Part 2 - RocksetGPT
<img width="527" alt="Screenshot 2024-03-28 at 12 06 59 AM" src="https://github.com/sofia099/RocksetGPT/assets/59860423/9687fa37-1933-4aad-884e-4ea9674822e0">

To build RocksetGPT, we need an OpenAPI spec of all the endpoints we want our custom GPT to access. You can use the spec you created in Part 1 or use the spec in this repository. Let's build a RocksetGPT:

1. Under "Explore GPTs" in the ChatGPT console, select "Create" in the upper right corner.
2. Give your GPT a name, description, and (optionally) a profile image.
3. Give your GPT a set of instructions (prompts):
    ```
    You are a data engineer with access to our company's Rockset account. Rockset is a cloud-native database. As a data engineer, you have the ability to use any of the endpoints provided in your Actions. You also have a list of SQL functions in your Knowledge. When the user asks you to write SQL, you can only use functions in this file. Everytime you write SQL, verify that the functions you are using exist in the functions.txt file. You can ONLY use functions in this file. These are all functions that are compatible with Rockset.
    ```
  
4. Give your GPT knowledge by uploading files to its Knowledge bank. For the RocksetGPT, let's upload a `functions.txt` file that has a list of Rockset SQL Functions and their description which can be copied from the Rockset SQL Reference: https://docs.rockset.com/documentation/reference/summary
5. Enable the "Code Interpreter" built-in capability so that it can interpret the functions we just uploaded and write its own queries.
6. Click "Create New Action".
7. Add an Authentication method. Rockset uses a _custom_ ApiKey authentication method. To implement this correctly, select/type the following:
    - Authentication Type = `API Key`
    - API Key = `ApiKey YOUR_API_KEY`
    - Auth Type = `Custom`
    - Custom Header Name = `Authorization`
8. Add the OpenAPI spec under `Schema`.
9. Test and you're done! Test individual endpoints to ensure the response is correct.

## Debugging
Debugging a no-code solution can be frusturating. Here are some tips I uncovered:
- Network connectivity issue means there is an issue with OpenAI's client code connecting to Rockset's API. Usually a result from incorrectly setting up your Authentication or wrong server url.
- Invalid parameter keyword indicates there is an issue with the schema. Ensure that the requestBody does not contain any `$ref`.
- Since you have to explictly define the authentication method, avoid including security schemas in the OpenAPI spec as this can sometimes cause issues (especially if you have custom authentication methods). OpenAI's client code will automatically create the security schema internally.
- If an endpoint is still not working, ask GPT for the curl request its sending and test it yourself to see if its incorrectly reading the schema.
- Use "gentle parenting" with your GPT. Ask "why did you do what you did" and "how can we prevent you from doing that again".
- Be explicit and clear in your instructions and requests - don't make any assumptions.
- If you are still seeing issues, try using `ActionsGPT` in OpenAI to build an example to see what's missing. You can provide ActionsGPT with an example curl request and response and it will output the spec.

### Interested in learning more?
Check out this recording of a workshop I hosted on Custom GPTs: <br />
TBD

Check out these slides from said workshop: <br />
https://docs.google.com/presentation/d/1TVqCDd8YQxQAxosvT6-ZCX4ERidTAAklQGRiBGnnCto/edit#slide=id.g2c731a7967e_0_0
