# Guidance for Q&A System with Similarity Search-based Retrieval Augmented Generation (RAG) on AWS

1. [Overview](#overview)
    - [Architecture](#architecture)
    - [Cost](#cost)
2. [Prerequisites](#prerequisites)
    - [Operating System](#operating-system)
3. [Deployment Steps](#deployment-steps)
4. [Running the Guidance](#running-the-guidance)
5. [Next Steps](#next-steps)
6. [Cleanup](#cleanup)
7. [Notices](#notices)
8. [Known Issues](#known-issues)
9. [Authors](#authors)

## Overview
Amazon DocumentDB provides native vector search capability to perform similarity search.  In this guidance, we provide a step-by-step guide with all the building blocks for creating an enterprise ready RAG application such as a question answering(Q&A) system. We use a combination of different AWS services including [Amazon Bedrock](https://aws.amazon.com/bedrock/), an easy way to build and scale generative AI applications with foundation models. We use [Titan Text](https://aws.amazon.com/bedrock/titan/) for text embeddings and [Anthropic's Claude on Amazon Bedrock](https://aws.amazon.com/bedrock/claude/) as our LLM and Amazon DocumentDB as our vector database. We also demonstrate integration with open-source frameworks such as LlamaIndex for interfacing with all the components.

### Architecture

This architecture diagram shows how to handle user queries to provide domain-specific responses effectively. It enhances a Foundation Model on Amazon Bedrock using a RAG approach, leveraging Amazon DocumentDB vector search and LlamaIndex to retrieve relevant data.

![Architecture](assets/Arch.PNG)

### How It Works

The application follows these steps to provide responses to your questions:

1. The new external data, which lies outside of the LLM's original training data, can come from various sources such as APIs, databases, or document repositories.
2. Data preprocessing to remove inconsistencies and errors, splitting large documents into manageable sections, and chunking the text into smaller, coherent chunks for easier processing. 
3. Generate text embeddings for relevant data using the Titan text embedding models on Amazon Bedrock.
4. Notebook fetches credentials from AWS Secrets Manager to connect to Amazon DocumentDB
5. Create vector search index and load the generated text embeddings along with other relevant information into a DocumentDB collection.
6. User submits a query for finding relevant answers and system receives a natural language question from the user
7. The user’s question is transformed into a vector embedding using the same embedding model on Amazon Bedrock that was used during data ingestion workflow.
8. Fetches credentials from AWS Secrets Manager to connect to Amazon DocumentDB
9. Perform a similarity search in the Amazon DocumentDB collection using the query embedding. The search retrieves the most relevant documents or document segments based on their proximity to the query vector in the embedding space
10. The user query and the retrieved relevant information are provided to the LLM, which utilizes both the new data and its training knowledge to generate more accurate responses

Step 7-10 can be simplified by using LlamaIndex - an opensource orchestration framework for RAG . LlamaIndex works as query engine and  enables the use of natural language to retrieve relevant contextually appropriate information from a vector search index in Amazon DocumentDB. This retrieved information, along with the user's question, is then provided to the LLM model to generate more accurate and informed responses.

### Cost

_You are responsible for the cost of the AWS services used while running this Guidance. We recommend creating a [Budget](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html) through [AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/) to help manage costs. Prices are subject to change. For full details, refer to the pricing webpage for each AWS service used in this Guidance._

### Sample Cost Table

The following table provides a sample cost breakdown for deploying this Guidance with the default parameters in the US East (N. Virginia) Region for one month.

| AWS service  | Dimensions | Cost [USD/month] |
| ----------- | ------------ | ------------ |
| Amazon DocumentDB Instance Based Cluster | Standard Cluster Configuration, 1 X Instance type (db.r6g.large), Storage (10 GB), I/Os (2 millions), Backup 1 Day | $193.50 |
| Amazon Bedrock - Titan Text Embeddings model | Number of Input tokens (100 million per month) | $10.00 |
| Amazon Bedrock - Anthropic Claude | Number of Input tokens (100 million per month) | $10.00 |
| Amazon SageMaker | Storage (General Purpose SSD (gp2)), Instance name (ml.t3.large), Number of data scientist(s) (1), Number of On-Demand Notebook instances per data scientist (1) | $72.28 |
| AWS Secrets Manager | 1 Secret, 1 million requests/month | $5.40 |

## Prerequisites

1. [Amazon Bedrock](https://aws.amazon.com/bedrock/) requires you to request access to its foundational models as a pre-requisite before you can start invoking the model using Bedrock APIs.Below we will configure [model access](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html) in Amazon Bedrock in order to build and run generative AI applications. Amazon Bedrock provides a variety of foundation models from several providers such as AI21 Labs, Anthropic, Cohere, Meta, Stability AI, and Amazon.

#### Amazon Bedrock Setup Instructions
- In the [AWS Console](https://aws.amazon.com/console/), select the Region from which you want to access Amazon Bedrock.
- For this guidance , we will be using the `us-west-2` region.
  ![](source/01_RetrievalAugmentedGeneration/01_QuestionAnswering_Bedrock_LLMs/static/Amazon_Bedrock_Region.png)

- Search for Amazon Bedrock by typing in the search bar on the AWS console.
  ![](source/01_RetrievalAugmentedGeneration/01_QuestionAnswering_Bedrock_LLMs/static/Bedrock_Console.png)

- Expand the side menu with three horizontal lines (as shown below), select Model access and click on Enable specific models button.
  ![](source/01_RetrievalAugmentedGeneration/01_QuestionAnswering_Bedrock_LLMs/static/Bedrock-Expand.png)
  ![](source/01_RetrievalAugmentedGeneration/01_QuestionAnswering_Bedrock_LLMs/static/Bedrock_Model_Access.png)

- For this guidance, we'll be using Anthropic's Claude 3 models as LLMs and Amazon Titan family of embedding models. Click Next in the bottom right corner to review and submit.
  ![](source/01_RetrievalAugmentedGeneration/01_QuestionAnswering_Bedrock_LLMs/static/Bedrock_Check_Model_Access.png)
  ![](source/01_RetrievalAugmentedGeneration/01_QuestionAnswering_Bedrock_LLMs/static/Bedrock_Submit_Model_Access.png)
  
- You will be granted access to Amazon Titan models instantly. The Access status column will change to In progress for Anthropic Claude 3 momentarily. Keep reviewing the Access status column. You may need to refresh the page periodically. You should see Access granted shortly (wait time is typically 1-3 mins).
  ![](source/01_RetrievalAugmentedGeneration/01_QuestionAnswering_Bedrock_LLMs/static/Bedrock_Model_Access_In_Progress.png)
  ![](source/01_RetrievalAugmentedGeneration/01_QuestionAnswering_Bedrock_LLMs/static/Bedrock_Access_Granted.png)

> [!Note]
> Now you have successfully configured Amazon Bedrock.

2. We recommend using Mozilla Firefox as the preferred browser for running the AWS Cloud9 Console. If you don't already have it, you can download it from [Mozilla Firefox](https://www.mozilla.org/en-US/firefox/new/).




## Notices

*Customers are responsible for making their own independent assessment of the information in this Guidance. This Guidance: (a) is for informational purposes only, (b) represents AWS current product offerings and practices, which are subject to change without notice, and (c) does not create any commitments or assurances from AWS and its affiliates, suppliers or licensors. AWS products or services are provided “as is” without warranties, representations, or conditions of any kind, whether express or implied. AWS responsibilities and liabilities to its customers are controlled by AWS agreements, and this Guidance is not part of, nor does it modify, any agreement between AWS and its customers.*

## Known Issues

 
### License

The Q&A System with Similarity Search-based Retrieval Augmented Generationn is released under the [MIT-0 License](https://spdx.org/licenses/MIT-0.html).

## Authors
- Gururaj Bayari
- Anshu Vajpayee

### Contribution
This repository is intended for educational purposes and does not accept further contributions. Feel free to utilize and enhance the app based on your own requirements.
    
