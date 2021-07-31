# Declarative pipeline for Demo

# Prerequisites

## Plugins required :- 
1. git -> for git checkout.
2. Http Requests -> for REST Api call.
3. Pipeline Utility Steps -> For reading JSON file(readJSON), zip files(zip) and unzip files(unzip).
4. Blue Ocean -> for visualization.

## Note -> This pipeline can run on both linux & windows agent.
## How to run :-

1. Create a pipeline item in Jenkins

![tempsnip](https://user-images.githubusercontent.com/14027267/127741092-ee3ad1d1-f1cf-40e1-b447-45784832b820.png)

2. Configure the pipeline to use the Jenkinsfile present in this repository.

3. Click on "build now" to run the pipeline. The First run will create necessary parameters.

4. The pipeline can be visualized in the Blue Ocean.

![image](https://user-images.githubusercontent.com/14027267/127741532-37734523-e4ff-4b01-96e4-f532ce79a156.png)




