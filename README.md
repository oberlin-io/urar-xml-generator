# urar-xml-generator
Inputs copied web browser tables, applies logic and math, and replaces dummy values in XML -- all in Google Sheets -- ready for import to software.

The purpose of this project was to replace a series of individual copy-and-paste data entries, case changes, math calculations, and basic if-then logic with a handful of entire web tables copy-and-pastes and a simple XML save and import.

The project is done completely on Google Sheets, and I'll save us all from the details of the context, as this could work for any situation in which one needs to take a significant amount of data from a browser into an XML, JSON, or CSV format for import into other software.

## XML Skeleton
The key initial setup, only done once, involves taking a skeleton XML document, exported from the said software, and replacing the values with a dummy variable, eg "var!" -- anything that would be highly unlikely to appear elsewhere in the XML doc.

Working with this XML document in a text editor, create a new line for every element or attribute that has a dummy variable. Then copy that XML text into a Google Sheet column. [Check my XML document for example](https://github.com/oberljn/urar-xml-generator/blob/master/skeleton.xml). Every "var!" has its own row. Technically, closing tags could be grouped together on one row, but I tried to atomize as much as I could for extensibility.

## Dummy Variable Replacement
Given that the XML is in column A, let column B be the input column and let C be the XML output. For every row in C, use the following formula:

```
REGEXREPLACE(A2, "var!", B2)
```

The regular expressions replace function takes the XML line in A2 and replaces the dummy variable with B2's value.

## Getting the Data

## Logic and Math
