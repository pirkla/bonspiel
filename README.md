# Bonspiel
Search and view sample curl commands for Jamf Pro endpoints gathered from the swagger schema.  

This is a bit of a mess right now, but I'm currently working on cleaning it up and will likely drastically change it.
This is meant as a proof of concept for a larger project, but it is pretty neat as a standalone tool. I'm throwing it here for reference and anyone that wants to check it out. 

## How to use
1. Download and unzip the repository
2. Run it via python3 ```python3 ~/Downloads/bonspiel-master/bonspiel.py```  
Check -h for usage.  
Example command: ```python3 ~/Downloads/bonspiel-master/bonspiel.py computers```  
Example command with sample: ```python3 ~/Downloads/bonspiel-master/bonspiel.py computers/id -s```
