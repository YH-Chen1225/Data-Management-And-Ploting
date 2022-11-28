# Exercise Data Management and Plotting

This excercise is done by MosesChen, GeweiCao, ZihanYang, YingyuWu, XingyueWang
## General Notes


- The runtime of the solution will strongly depend on the Computer you are using, but even more on the code you write. If you often loop over each item in a pandas Series or DataFrame, the runtime explodes. In those cases you should look for a built-in pandas function that does the job.

- As a general rule, modified datasets should always be saved in a different directory than original data. Never ever overwrite an original data file! During this assignment, all generated output should be saved in a folder called `bld`. In some tasks, we ask you explicitly to save datasets. If you want to save more often, you are free to do so. 

- This assignment is not easy and depending on your previous experience it will take quite some time to complete it. However, we would never ask you to perform boring or repetitive tasks by hand. Whenever you find yourself typing long lists of variable names or you have several lines of code that look very similar, you can be sure that there is a better way. 

- Most of the tasks here have very short and elegant solutions. Our example solution has between 4 and 20 lines of code per task. You can use those numbers as a reference point for your own solution. If you have much longer solutions your are probably doing work that the pandas developers have already done for you. It can save you a lot of time to read up on pandas in [Wes McKinney's book](https://www.oreilly.com/library/view/python-for-data/9781491957653/) or the [online documentation](http://pandas.pydata.org/pandas-docs/stable/). 

- All previous comments about following the python style guides, writing good commit messages, not having generated files under version control and sticking to the file and folder names we ask you to use, still apply. In particular the rules for docstrings still apply

- In the later sections of this assignment you need to create plots using plotly. You will
be introduced to plotly in lecture 5.
    
- This time there are not pre-commit hooks and no tex file. You only work in a jupyter notebook. You can either open the notebook in VS Code, in Jupyterlab or as a standalone
notebook in your browser. 


## Background

In this assignment you will do data management for the paper *Estimating the Technology of Cognitive and Noncognitive Skill Formation* by Cunha, Heckman and Schennach (CHS), Econometrica, 2010. 

Doing the complete data management of such a complicated project is not possible in one assignment (it often takes weeks or months). Therefore, you will only work with a small subset of the variables needed to replicate the paper. Moreover, we will save you some of the most painful steps by providing a pre-processed version of the dataset and csv files that will help you to harmonize variable names between panel waves. 

We will focus on the Behavior Problem Index that is used to measure non-cognitive skills. This index has the subscales antisocial behavior, anxiety, dependence, headstrong, hyperactive and peer problems. Here is an [overview](https://www.nlsinfo.org/content/cohorts/nlsy79-children/other-documentation/codebook-supplement/appendix-d-behavior-proble-0).

The assignment repository contains a file called `src/original_data/original_data.zip`, with four files in it:

- `BEHAVIOR_PROBLEMS_INDEX.dta`: Contains the main data you will work with. It is in wide format and the variable names are not informative. Moreover, the names do not contain the survey year in which the question was asked.
- `bpi_variable_info.csv`: Contains information that will help you to decompose the main dataset into datasets for each year and to rename the variables such that the same questions get the same name across periods. In a real project you would have to generate this information yourself.
- `BEHAVIOR_PROBLEMS_INDEX.cdb`: The codebook of the dataset. If you have any questions about the data, the answers are probably in the codebook.
- `chs_data.dta`: The data file used in the original paper by Cunha Heckman and Schennach.
## Tasks

1. Clone this repository to your machine, if you have not already done so. Create a notbeook in the `src` directory called `solution.ipynb`. All of your solution will be contained in that notebook. Commit.
2. Unzip the the file `src/original_data/original_data.zip`. This [stackoverflow post](https://stackoverflow.com/questions/3451111/unzipping-files-in-python) tells you how. Since the unzipped files are generated, they should not be under version control and be stored in the `bld` directory. 
3. You have learned in the class that it is always good to minimize state. The main way of achieving this is to put as much code as possible into pure functions. But of course you cannot make all of your data management functions pure. After all, at some point you need to load data from disk or save it, which is considered a side effect. In jupyter notebooks you get another form of state: Cells can be executed several times. Briefly discuss the following questions (in the solution notebook). 
    - How should functions that generate a variable look like. What arguments should they take and what should they return?
    - How can you avoid that executing a cell several times leads to a different result than executing it once?
4. Load `chs_data.dta` into a pandas DataFrame. Transform it into a new dataframe that differs from the old one in the following aspects:
    - It has the colum `"mom_id"` which of dtype integer and does not have the original column `"momid"`
    - It does not contain observations where `"year"` is less than 1986 or above 2010 or where the year number is odd. 
    - The index is set to `["child_id", "year"]` (both of dtype integer).
5. Load the `bpi` dataset, clean and transform it. This means that you generate a new dataset that differs from the old one by the following aspccts. You can vary the order in which you achieve these tasks as you wish.
    - Missing values are coded as [pandas missings](https://pandas.pydata.org/pandas-docs/stable/user_guide/missing_data.html) instead of negative numbers. 
    - The index is the same as in the chs data. `"child_id"` can be generated from `"C0000100"` 
    - All variables have readable names from `bpi_variable_info.csv`
    - Only keep variables that are present in `bpi_variable_info.csv`
    - The data is in long format
6. Merge the datasets from the two previous task. Keep all observations that are in the chs dataset but drop all that are only in the bpi dataset. Before you merge, check if there are any overlaps in column names between the two datasets. If so, solve them by renaming those columns before merging. 
7. Calculate scores for each subscale of the behavioral problems index (bpi) by averaging the items of that subscale. The dataset contains surprises such as categorical variables where you need numerical ones, variables that take values you did not expect or category labels that are similar but not identical across variables. Try to find good solutions for each of them. Also, use the following conventions:
    - The answers 'sometimes true' and 'often true', are counted as 1, 
    - 'not true' is counted as zero. 
    - Standardize the scores to mean 0 and variance 1 for each age group. 
8. Save the dataset you generated in the previous task in `.pickle` format in the `bld` folder under a suitable file name.
9. Make a [regression plot](https://plotly.com/python/ml-regression/) for each subscale that show how your score relates to the corresponding score in the chs data. Note that in the chs data missings are sometimes coded as -100. The names of their score relate to your names as follows: 
    ```python
    {
        "antisocial": "bpiA",
        "anxiety": "bpiB",
        "headstrong": "bpiC",
        "hyperactive": "bpiD",
        "peer": "bpiE",
    }
    ```
    The dependence scale has no counterpart in the chs data. If you did everything correctly you should see a strong but not perfect negative correlation in each case.
10. The latent factor model behind the technology of skill formation implies that all items that make up the behavior problems index should be positively correlated, with particularly high correlations between the items of one subscale. Make a [heatmap](https://plotly.com/python/heatmaps/) of the correlation matrix of the items from at least three subscales. Order the items such that items that belong to the same subscale are closer together. Choose a color scale in which it is easy to distinguish negative, positive and zero correlations without looking at numbers. 
11. Save the plots under suitable filenames in `.png` format in the `bld` folder.
12. Once you are satisfied with your solution, merge it into the master branch of your repository. Then push to the central server.
