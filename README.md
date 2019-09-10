**[Automating Data Delivery using the OData Endpoint in ODK
Central]{dir="ltr"}**

[]{dir="ltr"}

**[Overview]{dir="ltr"}**

[]{dir="ltr"}

[[[ODK
Central]{.underline}](https://docs.opendatakit.org/central-intro/)
includes an [[OData]{.underline}](https://www.odata.org/) endpoint that
can be accessed by tools and languages that support RESTful web
services. OData has been shown to work with Tableau and PowerBI,
allowing users to download survey submissions for immediate analysis.
But what about other scenarios? A common task is to retrieve ODK
submissions, extract the survey data and load it into a data portal or
relational database. ODK Central can support this use case as
well.]{dir="ltr"}

[]{dir="ltr"}

[The purpose of this article is to show how this can be easily
accomplished using an open source data integration tool (Apache Kettle).
We access submissions from a survey hosted on the ODK Central sandbox
server, transforming the JSON document into tabular rows, then loading
the rows into a dataset stored in a CKAN data portal. The workflow is
implemented using a small number of visual components and can be freely
used by anyone in the ODK Community. Additional scenarios are discussed,
and links to additional resources are provided at the end.]{dir="ltr"}

[]{dir="ltr"}

**[About Kettle]{dir="ltr"}**

[]{dir="ltr"}

[[[Kettle]{.underline}](http://www.ibridge.be/) is an open source tool
for extracting, transforming and loading data. The design paradigm is
based on the concept of components ("steps") that are connected to form
a data pipeline (aka a *directed graph*). Data flows from component to
component as defined in the graph; this fulfils the logic of the
transformation. Each component implements a specific type of
functionality such as calling a web service, performing a lookup,
calculating values or writing to a file or database table. With a
library of over 100 built-in steps, Kettle is capable of handling
sophisticated scenarios.]{dir="ltr"}

[]{dir="ltr"}

[The Kettle engine runs as an embedded service within a graphical
designer (Spoon). To simplify configuration management and to enforce
security, the designer can be deployed as a web application (WebSpoon).
Kettle also runs as a "headless" process on a server to support
production deployments in a cloud environment. A companion service
implements job orchestration. This allows a set of transformations to be
executed with dependency checking and error handling.]{dir="ltr"}

[]{dir="ltr"}

**[About CKAN]{dir="ltr"}**

[]{dir="ltr"}

[[[CKAN]{.underline}](https://ckan.org/) is a tool for creating \"open
data\" websites. Think of it as a content management system (like
WordPress) but for data rather than pages and blog posts. It allows
organizations to manage and publish collections of structured data. CKAN
is particularly useful when data needs to be shared by a group of
organizations. It is used by a significant number of national and local
governments, research institutions, and other organizations, e.g.
Nigeria\'s [[GRID3]{.underline}](http://grid3.org/) data portal and the
[[Humanitarian Data Exchange]{.underline}](https://data.humdata.org/)
(HDX), and [[many
others]{.underline}](https://ckan.org/about/instances/).]{dir="ltr"}

[Once data has been published, users can use it\'s search features to
browse and download the data they need. They can also preview it using
maps, graphs and data. CKAN stores metadata about datasets and
resources, and also offers a powerful API (used in our solution) that
allows third-party applications and services to be built around
it.]{dir="ltr"}

[]{dir="ltr"}

[Like Kettle, CKAN is open source software with an active developer
community that has contributed more than 200 plugins. You can test drive
\"vanilla\" CKAN using the [[online
demo]{.underline}](https://demo.ckan.org/) instance. This solution uses
a CKAN instance hosted by
[[Amplus.io]{.underline}](https://amplusdata.io/).]{dir="ltr"}

[]{dir="ltr"}

**[Workflow]{dir="ltr"}**

[]{dir="ltr"}

[Figure 1 illustrates how our use case appears in Kettle. One benefit is
immediately clear -- the transformation logic is easy to understand.
Only five steps are required to download the submissions from ODK
Central, to parse them and to load them into CKAN. As indicated by the
arrows, the data flows from left to right. Let's look at each of these
steps in more detail.]{dir="ltr"}

[]{dir="ltr"}

![](media/image1.png){width="6.5in"
height="0.7118055555555556in"}[]{dir="ltr"}

[Figure 1]{dir="ltr"}

[]{dir="ltr"}

**["Set the URL"]{dir="ltr"}**

[]{dir="ltr"}

[The transformation starts with a *Data Grid* step. This step is used
anywhere we want to generate arbitrary rows of data. For example, it can
be used to create reference lists such as a custom calendars or lookup
tables. In our case we use it to define the value of the ODK Central
sandbox's OData endpoint URL. Under the *Meta* tab we see that a field
called *url* is being created (Figure 2a). Under the *Data* tab we
assign the ODK Central endpoint to the *url* field (Figure 2b). This
value will get passed as an *output field* to the downstream step which
will treat it as input.]{dir="ltr"}

[ ]{dir="ltr"}

![A screenshot of a cell phone Description automatically
generated](media/image3.png){width="6.5in"
height="1.5993055555555555in"}[]{dir="ltr"}

[Figure 2a]{dir="ltr"}

[]{dir="ltr"}

![A screenshot of a cell phone Description automatically
generated](media/image2.png){width="6.5in"
height="1.58125in"}[]{dir="ltr"}

[Figure 2b]{dir="ltr"}

[How did we determine which URL to use? ODK Central makes this easy.
Under the *Submissions* tab of any survey you'll see the option to
*Analyze via OData*. Simply click on this button to discover the URL
(Figure 2c):]{dir="ltr"}

![A screenshot of a cell phone Description automatically
generated](media/image6.png){width="6.255208880139983in"
height="2.565452755905512in"}[]{dir="ltr"}

[Figure 2c]{dir="ltr"}

**["Get Submissions from ODK Central"]{dir="ltr"}**

[ ]{dir="ltr"}

[To retrieve our survey data from the OData endpoint, we use a Kettle
step called the *REST Client*. This step is not specific to ODK -- it
can be used with any well-formed REST endpoint. In our case we make a
single call to the endpoint and it returns a JSON document containing
the survey collection. The connection to ODK Central is a simple "fill
in the blanks" exercise. In Figure 3a, we see that the OData endpoint
URL is defined in the field *url* (passed to us by the upstream step)
and that the HTTP method is GET. The JSON document returned by the OData
REST call is stored in a field called *result*.]{dir="ltr"}

[]{dir="ltr"}

![A screenshot of a cell phone Description automatically
generated](media/image5.png){width="6.405142169728784in"
height="4.515625546806649in"}[]{dir="ltr"}

[Figure 3a]{dir="ltr"}

[]{dir="ltr"}

[]{dir="ltr"}

[We need to provide credentials to ODK Central. ODK Central supports
basic authentication, so we provide values for the *HTTP Login* and
*HTTP Password* parameters (Figure 3b).]{dir="ltr"}

![A screenshot of a cell phone Description automatically
generated](media/image4.png){width="6.5in" height="4.5875in"} [
]{dir="ltr"}

[Figure 3b]{dir="ltr"}

[]{dir="ltr"}

[Note the use of a variable called *\${ODK CENTRAL.USER}*. Kettle allows
administrators to define variables that are referenced dynamically at
runtime. This supports complete lifecycle testing, simplifies
maintenance and improves security.]{dir="ltr"}

[]{dir="ltr"}

**["Extract the Survey Fields"]{dir="ltr"}**

[]{dir="ltr"}

[Once the transformation has executed the OData endpoint call, the
*result* field contains our survey collection. The *JSON Input* step
allows us to specify the key/value pairs in the *result* document that
correspond to the survey questions, and to map the values to new fields
that appear in the output stream (Figure 4). Kettle supports JSONPath
expressions to allow for precise extraction of fields within the
submissions document's structure.]{dir="ltr"}

![A screenshot of a cell phone Description automatically
generated](media/image8.png){width="6.5in"
height="4.175in"}[]{dir="ltr"}

[Figure 4]{dir="ltr"}

**["Select Fields for CKAN"]{dir="ltr"}**

[]{dir="ltr"}

[It's not uncommon to want "friendly names\" for use in reports and
analyses. It may also be necessary to translate survey labels to a
different language. We can use the *Select Values* step to accomplish
this and to re-order or remove columns as needed (Figure 5).]{dir="ltr"}

![A screenshot of a cell phone Description automatically
generated](media/image7.png){width="6.5in"
height="4.934027777777778in"}[]{dir="ltr"}

[Figure 5]{dir="ltr"}

[]{dir="ltr"}

**["Write to CKAN"]{dir="ltr"}**

[]{dir="ltr"}

[We're now ready to write the survey answer records to CKAN. Unlike our
use of the general-purpose *REST Client* step with the OData endpoint,
the *CKAN Writer* step is specific to CKAN (Figure 6a). It\'s a
purpose-built step that interacts with the data portal using the CKAN
API. Special-purpose steps are useful when a data source has unique
characteristics. For example, CKAN is based on the concept of *datasets*
that contain *resources* and also uses an API key for authentication.
The Kettle step library can be extended with community-developed
plug-ins; the CKAN Writer step is a good example.]{dir="ltr"}

![](media/image10.png){width="6.5in"
height="7.888888888888889in"}[]{dir="ltr"}

[Figure 6a]{dir="ltr"}

[]{dir="ltr"}

[]{dir="ltr"}

[Once we've executed the transformation we can preview the survey
answers in the CKAN portal (Figure 6b), proving that the data delivery
process was automated end-to-end. With sufficient data it is also
possible to use CKAN\'s available
[[visualizations]{.underline}](https://docs.ckan.org/en/2.8/maintaining/data-viewer.html).]{dir="ltr"}

[]{dir="ltr"}

![](media/image9.png){width="6.5in" height="2.5in"}[]{dir="ltr"}

[Figure 6b]{dir="ltr"}

[]{dir="ltr"}

**[Additional Scenarios]{dir="ltr"}**

[]{dir="ltr"}

[We chose a simple use case to illustrate how a small subset of Kettle
steps could be used to implement basic workflow. In the real world
we\'re likely to encounter additional requirements:]{dir="ltr"}

[]{dir="ltr"}

-   [[Loading to relational databases]{.underline}. In our example,
    loading to MySQL, PostgreSQL, SQL Server or another JDBC-compatible
    database would be as simple as replacing the *CKAN Writer* step with
    a *Table Output* step. It\'s even possible to write to multiple
    output destinations simultaneously.]{dir="ltr"}

-   [[Multi-value surveys (i.e. with questions that result in a
    variable-length array of answers)]{.underline}. This is a common
    scenario which Kettle can support by \"cascading\" JSON Input steps
    in a transformation. Multi-value fields are extracted in the first
    step (as JSON "sub-documents"), then parsed in a second
    step.]{dir="ltr"}

-   [[Merging submissions from related-but-different
    surveys]{.underline}. Surveys evolve over time and there is often a
    requirement harmonize semantically equivalent data. Since Kettle can
    dynamically create and alter fields, it's possible to merge data
    from multiple versions of a survey. Default and/or calculated values
    can be used with missing questions.]{dir="ltr"}

-   [[Capturing and recording metadata]{.underline}. Metadata is
    available directly in the submissions document, in the XLSForm and
    also in the runtime environment. Kettle can read, extract and
    correlate metadata from multiple sources to create detailed logs for
    the purpose of documentation and governance.]{dir="ltr"}

-   [[Enrichment with existing data]{.underline}. Survey often use
    drop-down lists to ensure consistent data quality. With a known set
    of values it\'s possible (using one or more key fields) to perform
    lookups against a database table or web service. More descriptive
    and/or additional values can be appended to produce an enriched data
    product.]{dir="ltr"}

-   [[Dynamically-generated XLSForms]{.underline}. Kettle\'s *Excel
    Writer* step provides an opportunity to generate XLSForms using data
    available at any point in a pipeline. The step can create new
    spreadsheets (with the option to use a template) and can update
    existing spreadsheets.]{dir="ltr"}

-   [[ODK-X]{.underline}. Although ODK-X does not support OData, it does
    have a REST API that allows a developer to retrieve survey results.
    Unlike ODK v1, ODK-X can return surveys submitted since a specific
    data, or surveys whose data elements have changed . See
    [[https://docs.opendatakit.org/odk-x/odk-2-sync-protocol/\#get-all-data-rows-api]{.underline}](https://docs.opendatakit.org/odk-x/odk-2-sync-protocol/#get-all-data-rows-api)
    as a starting point.]{dir="ltr"}

[**Summary**]{dir="ltr"}

[]{dir="ltr"}

[Our example has shown that it\'s possible to retrieve submissions from
ODK Central, extract fields from survey collections and load them as
rows into CKAN. *The solution stands out for its simplicity* - anyone
who can install a Java application on Windows, Linux or MacOS can
execute the workflow with minimal difficulty. With a modest orientation
it\'s possible for data-savvy analysts to adapt and extend the data
pipeline for additional use cases. Organizations with budget constraints
- and consortiums with the need to standardize on an open source stack -
can use Kettle with ODK Central to standardize post-survey processing
and to improve interoperability.]{dir="ltr"}

[]{dir="ltr"}

**[Try It Yourself:]{dir="ltr"}**

[]{dir="ltr"}

[Kettle (including the CKAN Datastore Writer Plugin) can be downloaded
at
[[http://kettle-builds.s3-website-eu-west-1.amazonaws.com/kettle-remix/]{.underline}](http://kettle-builds.s3-website-eu-west-1.amazonaws.com/kettle-remix/).
This solution was developed and tested using [[Kettle
8.2.0.7-719]{.underline}](http://kettle-neo4j-remix-8.2.0.7-719-remix.tgz)
on Ubuntu 18.04 LTS.]{dir="ltr"}

[]{dir="ltr"}

[CKAN can be downloaded at
[[https://ckan.org/download-and-install/]{.underline}](https://ckan.org/download-and-install/)]{dir="ltr"}

[]{dir="ltr"}

[A simple transformation for testing Kettle connectivity to]{dir="ltr"}

[]{dir="ltr"}

**[CKAN Datastore Writer Plugin \-- Attribution]{dir="ltr"}**

[]{dir="ltr"}

[Initial Release:]{dir="ltr"}

[]{dir="ltr"}

-   [Caio Moreno de Souza:
    > [[https://github.com/caiomsouza/CKAN-DataStore-Writer-for-Pentaho-Data-Integration]{.underline}](https://github.com/caiomsouza/CKAN-DataStore-Writer-for-Pentaho-Data-Integration)]{dir="ltr"}

[]{dir="ltr"}

[Enhancements:]{dir="ltr"}

[]{dir="ltr"}

-   [OpenGov:
    > [[https://github.com/OpenGov-OpenData/CKAN-DataStore-Writer-for-Pentaho-Data-Integration]{.underline}](https://github.com/OpenGov-OpenData/CKAN-DataStore-Writer-for-Pentaho-Data-Integration)]{dir="ltr"}

-   [KnowBI:
    > [[https://github.com/knowbi/knowbi-kettle-ckan-step]{.underline}](https://github.com/knowbi/knowbi-kettle-ckan-step)]{dir="ltr"}

[]{dir="ltr"}

**[Kettle Documentation]{dir="ltr"}**

[]{dir="ltr"}

> **[Pentaho Kettle Solutions: Building Open Source ETL Solutions with
> Pentaho Data Integration]{dir="ltr"}**
>
> [by [[Matt
> Casters]{.underline}](https://www.amazon.com/s/ref=dp_byline_sr_book_1?ie=UTF8&field-author=Matt+Casters&text=Matt+Casters&sort=relevancerank&search-alias=books)
> (Author), [[Roland
> Bouman]{.underline}](https://www.amazon.com/s/ref=dp_byline_sr_book_2?ie=UTF8&field-author=Roland+Bouman&text=Roland+Bouman&sort=relevancerank&search-alias=books)
> (Author), [[van Dongen,
> Jos]{.underline}](https://www.amazon.com/s/ref=dp_byline_sr_book_3?ie=UTF8&field-author=van+Dongen%2C+Jos&text=van+Dongen%2C+Jos&sort=relevancerank&search-alias=books)
> (Author)]{dir="ltr"}
>
> [Available on
> [[Amazon]{.underline}](https://www.amazon.com/Pentaho-Kettle-Solutions-Building-Integration/dp/0470635177/)]{dir="ltr"}
>
> []{dir="ltr"}
>
> **[Pentaho Data Integration Quick Start Guide: Create ETL processes
> using Pentaho]{dir="ltr"}**
>
> [by [[Maria Carina
> Roldan]{.underline}](https://www.amazon.com/s/ref=dp_byline_sr_book_1?ie=UTF8&field-author=Maria+Carina+Roldan&text=Maria+Carina+Roldan&sort=relevancerank&search-alias=books)
> (Author)]{dir="ltr"}
>
> [Available on
> [[Amazon]{.underline}](https://www.amazon.com/Pentaho-Integration-Quick-Start-Guide/dp/1789343321)]{dir="ltr"}

[]{dir="ltr"}
