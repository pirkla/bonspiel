#!/usr/bin/env python
import urllib3

import json
import re
import argparse
import os.path

import xml.etree.ElementTree as ET

# get all the endpoints from the schema
def getEndpoints(schema, keyword):
    resultArray = []
    for path in schema.get("paths"):
        if(re.search(keyword,path)):
            pathDict = {}
            callTypes = {}
            pathDict.update({"url":path})
            for callType in schema.get("paths",{}).get(path,{}):
                newObjDef = schema.get("paths",{}).get(path,{}).get(callType,{}).get("parameters",{})
                xmlSample = None
                if type(newObjDef) == list:
                    for listObj in newObjDef:
                        schemaDict = listObj.get("schema",None)
                        if type(schemaDict) == dict:
                            defLoc = schemaDict.get("$ref")
                            defSplit = defLoc.split("/")
                            xmlData = schema.get(defSplit[1]).get(defSplit[2])
                            root = xmlData.get("xml",{}).get("name",defSplit[2])
                            # realData returns the full xml rather than root post data which leaves out a lot of definitions. 
                            # Leaving it here in a comment for now
                            # realData = schema.get(defSplit[1]).get(root)
                            xmlSample = xmlSampleMaker(root,xmlData,schema)[1]
                callTypes.update({callType:{"sample":xmlSample}})
            pathDict.update({"callTypes":callTypes})
            resultArray.append(pathDict)
    return resultArray

# create an xml sample from the api schema
def xmlSampleMaker(root,myDict,schema):
    treeBase = ET.Element(root)
    loopXMLSubTree(myDict,treeBase,schema)
    stringTree = ET.tostring(treeBase)
    return treeBase, stringTree

# loop to be used with xmlSampleMaker. Should probably change the architecture to reflect that.
def loopXMLSubTree(myDict,xmlEl,schema):
    for subKey in myDict.get("properties",{}).keys():
        sample = myDict.get("properties",{}).get(subKey,{}).get("example") or ""
        doc = ET.SubElement(xmlEl,subKey)
        doc.text = str(sample)
        loopXMLSubTree(myDict.get("properties",{}).get(subKey,{}),doc,schema)
        loopXMLSubTree(myDict.get("properties",{}).get(subKey,{}).get("items",{}),doc,schema)

        defLoc= myDict.get("properties",{}).get(subKey,{}).get("$ref")
        if type(defLoc) != type(None):
            defSplit = defLoc.split("/")
            xmlData = schema.get(defSplit[1]).get(defSplit[2])
            root = xmlData.get("xml",{}).get("name",defSplit[2])
            realData = schema.get(defSplit[1]).get(root)
            loopXMLSubTree(realData,doc,schema)

# create a sample curl command from an endpoint, call type, and sample.
def sampleCurl(endpoint,callType,sample):
    data = ""
    headers = '-H "accept: application/xml" '
    if callType == "put" or callType == "post" :
        if sample == None:
            sample = "Uh oh, I couldn't find a sample but there should be one. There's a few that do this; I'm looking into it."
        data = "-d " + '"' + str(sample) + '"'
        headers= '-H "accept: application/xml" -H "Content-Type: application/xml"'
    baseUrl = "https://sample.jamfcloud.com/JSSResource"
    catUrl = baseUrl + endpoint
    # consider making k optional since it bypasses trust
    curlInfo = "curl -ku 'user:password' -X " + callType +" " + catUrl + " " + data + " " + headers
    return curlInfo

def main():
    parser = argparse.ArgumentParser(description="Find endpoints and show examples.\nExample command: %(prog)s computer -s",
        usage='use "%(prog)s --help" for more information',
        formatter_class=argparse.RawTextHelpFormatter)
    parser.add_argument("keyword", help = "Search for this keyword in the Jamf Pro api endpoints")
    parser.add_argument("-s", "--show-sample", help="Show samples for xml data", action="store_true")

    args = parser.parse_args()

    # this should be moved to its own function. It'll work for now.
    if args.keyword is not None:

        swaggerSchema = ""

        if os.path.exists('/tmp/jamfschema.json'):
            with open('/tmp/jamfschema.json') as myjson:
                swaggerSchema = json.load(myjson)
        else:
            http = urllib3.PoolManager()
            r = http.request("GET","https://developer.jamf.com/smartdocs/classic-api/openapi.json")
            swaggerSchema = json.loads(r.data.decode('utf-8'))
            with open('/tmp/jamfschema.json','w') as outfile:
                json.dump(swaggerSchema,outfile)
        
        results = getEndpoints(swaggerSchema,args.keyword)
        for result in results:
            print ("\n**********"), (result.get("url")),  ("**********")
            for call in result.get("callTypes"):
                print("    " + call)
                if args.show_sample:
                    callType = call 
                    endpoint = result.get("url")
                    sample = result.get("callTypes",{}).get(call,{}).get("sample", "")
                    print(sampleCurl(endpoint,callType,sample))

if __name__ == '__main__':
    main()
