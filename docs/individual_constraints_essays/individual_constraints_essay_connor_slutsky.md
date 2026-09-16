# Design Constraints Essay
Senior Design I  
9/15/26  
Project: ASL to Text Translator  
Team: Connor Slutsky, Isaac Dowdy, Braden Monnin  

## Economic  
We want to keep spending to a minimum on this project, as I’m not aware of any outside funding for senior computer science capstones, so all spending would have to come out of our pockets. Thus, we’ll likely only use software, tools, and data that are free and/or open source, or the cheapest option if a free/open-source version isn’t available. For FOSS, this will require us to properly credit those developers if necessary, ask for permission if necessary, and to obey licenses. Additionally, we don’t anticipate that this project will be sold or be used for any monetary gain. 

## Diversity and Cultural
The first thing that we have to be conscious of is being correct, as none of us developers are familiar with ASL, so it wouldn’t be very appropriate for us to create a tool such as this without some oversight by someone who is familiar with ASL. This should reduce the chance of mistranslations and improve the usability of the tool for the intended user group – people that are learning ASL. For scope reasons, although there are many standardized sign languages, this tool will focus on ASL. We also don’t want the tool to recognize inappropriate words or fingerspelling, so some sort of filter may have to be implemented to prevent people from abusing or misusing the tool. We should also consider translating ASL to other spoken languages besides English, but doing so would probably require an outside translation tool (e.g. AI, Google Translate), which might not always be correct, and may also pose a privacy vulnerability if we inadvertently send these outside tools sensitive information – this would sacrifice Security for Diversity and Cultural expansion.   

## Security  
The most important constraint would be data privacy, as we will need to access a user’s camera to detect their ASL signs in order to translate it into spoken language, so we must not store this visual data excessively or for longer than necessary. And since we’re likely going to be dependent on FOSS software, we must understand the risks of FOSS software and use the most updated versions of that software to prevent security vulnerabilities. And lastly, if we have to depend on an existing AI model to translate the ASL data to spoken language output, we should be careful about what data we’re feeding it if it’s not a locally deployed model. We want to reduce the risk of the data being compromised in the hands of whoever’s deploying the model by feeding it only necessary data, if we have to feed it any at all.   


