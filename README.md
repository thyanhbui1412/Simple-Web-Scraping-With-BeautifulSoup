### Simple-Web-Scraping-With-BeautifulSoup

# Import libraries
from bs4 import BeautifulSoup
from urllib.request import urlopen 
import requests
import re
import pandas as pd

URL = "https://www.indeed.com/jobs?q=analytics&l=New+York&vjk=d17bf3f8f918cc5e"
r = requests.get(URL) 
c = r.text
soup = BeautifulSoup(c, "html.parser") #getting job postings

#look for td tag and taking attribute resultContent
x = soup.find_all("td", class_="resultContent") 

#find how many job postings we extracted
print(len(x)) 

# each job posting has a job position, company, and location
# some also include company website, company ratings and job salary
for item in x: 
    print(item) 
type(x)
#Functions to extract information: 

def extract_company(j):
    return j.find("span", class_ = "companyName").get_text()

def extract_position(j):
    return j.find("span", title = re.compile(".*?")).get_text()

def extract_salary(j):
    if j.find("span", class_ = "estimated-salary") is not None:
        return j.find("span", class_ = "estimated-salary").svg["aria-label"]
    # specifed Not provided rather than leave it as None data type
    # to fix error when writing csv file
    else:
        return "Not provided" 

def extract_location(j):
    return j.find("div", class_ = "companyLocation").get_text()

def extract_rating(j):
    if j.find("span", class_ = "ratingsDisplay withRatingLink") is not None: 
        return j.find("span", class_ = "ratingsDisplay withRatingLink").a['aria-label']
    # specifed Not provided rather than leave it as None data type
    # to fix error when writing csv file
    else:
        return "Not provided"
    
def extract_webpage(j):
    if j.find("span", class_="companyName").a is not None: 
        originalLink = "www.indeed.com"
        link = originalLink + j.find("span", class_="companyName").a["href"]
        return link
    else: 
        return "Not provided" 
jobs = {} #create empty dictionary

for job in x: 
    
    #call functions to fill dictionary
    
    company = extract_company(job)


    jobs[company] = {} #set company name to empty dictionary
    
    jobs[company]["Position"] = extract_position(job)
    
    jobs[company]["Salary"] = extract_salary(job)  
    
    jobs[company]["Location(s)"] = extract_location(job) 
    
    jobs[company]["Company Indeed Webpage"] =  extract_webpage(job)
    
    jobs[company]["Company Rating"] = extract_rating(job)
    
for k,v in jobs.items(): 
    print(k,v, sep="\n") #print to see elements of dictionary
    
#Construct Pandas DataFrame from dictionary 
jobs_df = pd.DataFrame(jobs)
jobs_df.head()

import json

#create new json file and copy content of jobs dictionary
with open("JobSearch.json", "w") as writeJSON: 
    json.dump(jobs, writeJSON)
    
import csv
   
#create new csv file and copy contents of jobs dictionary
with open("JobSearch.csv", "w") as toWrite:  
    writer = csv.writer(toWrite, delimiter = ",")
    writer.writerow(["Company","Position","Salary","Location(s)", "Company Indeed Webpage", "Company Rating"])
    for a in jobs.keys():
        writer.writerow([a, jobs[a]["Position"], jobs[a]["Salary"], jobs[a]["Location(s)"], jobs[a]["Company Indeed Webpage"], jobs[a]["Company Rating"]])
        
