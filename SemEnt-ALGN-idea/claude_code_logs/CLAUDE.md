- While writing code that references any other file in the directory, first review the referenced files first for corrected function names, inputs and outputs, and only then implement the reference.

- If the tasks include data loading and wrangling from external datasets, begin with first reviewing at least one entry from the dataset to ensure what column names and dataset structure exist and how should they be mapped for loading, preprocessing and wrangling.  
    - Along with reviewing the column names for the right mapping, please also print out the data to review what is required for successful experiment implementation. 
    - In the case of combining two columns for preprocessing, clearly consider the order in which these columns must be combined and log the decision. 

- All the code to be written should have detailed and extensive logging on terminal showcasing process and inputs for easy debugging. 

- All code must be run on a Modal App. Before any execution ensure that you have initialised modal correctly. Before writing any modal run file, please review the most recent documentation on: https://modal.com/docs/guide/apps and https://modal.com/docs/guide/project-structure. 
    - While setting up a Modal app, use deploy to create a persistent app, instead of running it ephemerally. 
    - Ensure that a persistent stable storage volume is used for all material related to the research idea. 

- Do not, ever, generate any TEST code. Implement code completely and to be production-ready always, unless explicitly specified otherwise in the experimental plan, hypotheses documents, or research idea text. 

- In the case of any tasks that take long in your estimation, only run them on Modal using the detach flag (-d, --detach). Here is more context: https://modal.com/docs/reference/cli/run. 

