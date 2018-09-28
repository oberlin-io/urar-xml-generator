# urar-xml-generator
Inputs tables or unstructured data, applies logic and math, and replaces dummy values in XML -- all in Google Sheets -- ready for import to software.

The purpose of this project was to replace a series of many individual copy-and-paste data entries, case changes, math calculations, and basic if-then logic with a handful of larger data copy-and-pastes and a simple XML save and import.

The project is done completely on Google Sheets, and I'll save us all from the details of the context, as this could work for any situation in which one needs to frequently take a significant amount of data from a browser or PDF and import it into some software, via XML, JSON, or CSV formats.

## XML Skeleton
The key initial setup, only done once, involves taking an exemplar skeleton XML document, exported from said software, and replacing the values with a dummy variable, eg "var!" -- anything that would be highly unlikely to appear elsewhere in the XML doc.

Working with this XML document in a text editor, create a new line for every element or attribute that has a dummy variable. Then copy that XML text into a Google Sheet column. [Check my XML document for example](https://github.com/oberljn/urar-xml-generator/blob/master/skeleton.xml). Every "var!" has its own row. Technically, closing tags could be grouped together on one row, but I tried to atomize as much as I could for extensibility. I also gave each row a sequence number to retain proper order in case the XML set is resorted.

## Dummy Variable Replacement
Given that the XML is in column A, let column B be the input column and let C be the XML output. For every row in C, use the following formula:

```
REGEXREPLACE(A2, "var!", B2)
```

The regular expressions replace function takes the XML line in A2 and replaces the dummy variable with B2's value. Important note: Numerical values will not replace nominal values, so in xml!B wrap the results in TO_TEXT().

## Getting the Input Data
Other tabs in the sheet house the particular data sets we wish to query into the XML tab's input column. Trying to avoid copying multiple pieces of data from one web page or PDF, the idea is to copy the entire web page or an entire table into its own Google Sheets tab.

HTML tables will copy nicely into Google Sheets, retaining the original table's structure. From the XML tab, a cell in column B (our XML values) can contain a QUERY() function. The following set of functions use regular expressions to check for a zip code in an address line and extracts and returns the zip code or else "Zip not found".

```
IF(
  REGEXMATCH(
    QUERY(base!A2:B, "SELECT B WHERE A = 'Site Address:'", 0),
    ".*\d\d\d\d\d.*"),
  REGEXEXTRACT(
    PROPER(
      QUERY(base!A2:B, "SELECT B WHERE A = 'Site Address:'", 0)), 
    "\d\d\d\d\d"),
  "Zip not found"
)
```

Regular expressions can be quite helpful for unstructured data not in a table. The copied data will likely be pasted into Google Sheets under one column, every line filling one row each. For example, in order to scrape data from a PDF document, whose table structure does not translate to Sheets, I created a regular expressions pattern to capture the needed data. The following captures a flood zone map panel number from a string like:

> Carrier Route: R003 Flood Zone Panel: 39151C0039E

```
REGEXEXTRACT(flood_data_tab!A1, "\d{5}.\d{4}.")
```

There are more complex sets of functions I employed in xml!B. Each cell is unique because it queries unique data and often requires unique math and logic. For example, the following does math on queried dates and based on time concatenates a particular nominal output. [Check this out for a description](https://github.com/oberljn/sheets-web-scrape).

```
IF(
  365 >= DAYS(
    TODAY(),
    QUERY(
      mls_dom!A2:I,
      "SELECT B WHERE F = 'New Listing' LIMIT 1",
      0
    )
  ),
  REGEXREPLACE(
    CONCATENATE(
      "NEOHREX MLS# ",
      QUERY(
        mls_dom!A2:I,
        "SELECT A WHERE F = 'New Listing' LIMIT 1",
        0
      ),
      " listed ",
      CONCATENATE(
        REGEXEXTRACT(
          QUERY(
            mls_dom!A2:I,
            "SELECT B WHERE F = 'New Listing' LIMIT 1",
            0
          ),
          "(\d\d/\d\d/)1\d"
        ),
        YEAR(
          REGEXEXTRACT(
            QUERY(
              mls_dom!A2:I,
              "SELECT B WHERE F = 'New Listing' LIMIT 1",
              0
            ),
            "\d\d/\d\d/\d\d"
          )
        ),
        " at ",
        QUERY(
          mls_dom!A2:I,
          "SELECT C WHERE F = 'New Listing' LIMIT 1",
          0
        ),
        ". ",
        "Currently shows ",
        QUERY(
          mls_dom!A2:I,
          "SELECT F WHERE A = '"&
            QUERY(
              mls_dom!A2:I,
              "SELECT A WHERE F = 'New Listing' LIMIT 1",
              0
            )
            &"' LIMIT 1",
          0
        ),
        " at ",
        QUERY(
          mls_dom!A2:I,
          "SELECT C WHERE A = '"&
            QUERY(
              mls_dom!A2:I,
              "SELECT A WHERE F = 'New Listing' LIMIT 1",
              0
            )
          &"' LIMIT 1",
          0
        ),
        "."
      )
    ),
    "\$",
    "\\$"
  ),
  "No current listings or listings in the past year reported in the NEOHREX multiple listing service."
)
```

Example output for the above code (Dollar signs needed escaped):

> NEOHREX MLS# 55555 listed 09/13/2018 at \$249,900. Currently shows Pending at \$249,900.
