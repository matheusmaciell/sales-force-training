
# ELF Scraper

This proof-of-concept scraper downloads the Salesforce event logs related to Lightning Reports through ELF (Event Log Files). 

These events are divided  into different chunks because of their volume. So, what the scraper does is to download all the necessary chunks of each Salesforce event of interest for our research, which is primarily to identify the non-performing reports over a time window of user interaction on them.

There are events of interest that are downloaded by default, such as Report, LightningPageView and LightningError, but it is possible to download other events like LightningInteraction (see EventTypes in the config.yaml file for more event types).


The Salesforce documentation used as reference was version 56, but we also have tested over version 57 without no problems.

You can check all the doc references in the Salesforce References section below.

---

## Installing requirements

The only requirement is to have Python (3.10.x) installed with some extra libraries. Once installed, open any command-line tool (e.g., CMD, PowerShell, etc) and run the following command.

```
cd scraper
python -m pip install -r requirements.txt
```

You should see message logs showing if it was successful.

## The Scraper process

The ELF Scraper is divided into 3 steps:

1. Download the chunks of the events of interest.
2. Build a set of datasets that can manage many-to-many relationships based on report execution and user session.
3. Update a database in the SQL Server the allow us to update our PowerBI dashboards on this server.

## Running ELF scraper

These following argument options are mandatory to successfuly run the ELF Scraper. 

| Option | Description |
|-|-|
| `-p` | Path to save the resulting datasets. |
| `-i` | List of optional lightning event types of interest. Check for `EventTypes` in `config.yaml`.|
| `-t` | List of hour range chunks. |
| `-r` | List of date range chunks.|
| `-v` | Sets verbose mode.|
| `--headless ` | Enables headless mode in Selenium.|
| `--skip-server-update` | Skips the step of updating server data.|
| `--skip-summarization` | Skip dataset creation for summarization if the user has the dataset or doesn't want to download it.|


Example:

```
python main.py -p "path\to\folder\" -i LightningInteration -r 2023-01-11 2023-01-13 -t 0-3 10 15-18 23
```


Only the specified dates and hours in `-t` and `-r` options will be downloaded.

##### Example:
```
2023-01-11, [00, 01, 02, 03, 10, 15, 16, 17, 18, 23]
2023-01-12, [00, 01, 02, 03, 10, 15, 16, 17, 18, 23]
2023-01-13, [00, 01, 02, 03, 10, 15, 16, 17, 18, 23]
```

The expected result is the generation of four different datasets for each type of Lightning event passed as argument.
- Report.csv 
- LightningPageView.csv
- LightningInteration.csv
- LightningError.csv
- UserSection.csv

Besides that, extra datasets will be created.
| Header | *Descprition* |
|-|-|
| Bridge.csv | Collection of execution keys to establish links between tables. |
| VennData.csv| Table for generating Vann diagrams. |
| VennSumm.csv| Extra information is added to a dashboard to provide a complete summary of various problematic aspects, allowing for effective presentation and summarization of multiple issues in a single interface. |

## Salesforce References

[- Report Event Type](https://developer.salesforce.com/docs/atlas.en-us.240.0.object_reference.meta/object_reference/sforce_api_objects_eventlogfile_report.htm?q=Report):  Report events contain information about what happened when a user ran a report. This event type includes all activity that's in the Report Export event type, plus more.
[- LightningPageView Event Type](https://developer.salesforce.com/docs/atlas.en-us.240.0.object_reference.meta/object_reference/sforce_api_objects_eventlogfile_lightningpageview.htm): Lightning Page View events represent information about the page on which the event occurred in Lightning Experience and the Salesforce mobile app.
[- LightningPageInteraction Event Type](https://developer.salesforce.com/docs/atlas.en-us.240.0.object_reference.meta/object_reference/sforce_api_objects_eventlogfile_lightninginteraction.htm): Lightning Interaction events track user actions in Lightning Experience and the Salesforce mobile app, such as the user clicking, tapping, or scrolling on a page.
[- LightningError Event Type](https://developer.salesforce.com/docs/atlas.en-us.240.0.object_reference.meta/object_reference/sforce_api_objects_eventlogfile_lightningerror.htm): Lightning Error events represent errors that occurred during user interactions with Lightning Experience and the Salesforce mobile app.
