# all_ntsb Splunk App
Splunk app to process NTSB Safety Data

*This is a work in progress.*

By far the easiest method is to install this app to a local Splunk Enterprise environment. But if you want, the instructions for Splunk Cloud are also included.

# Setup on local Splunk Enterprise
1. Create an index on your Splunk instance, I called mine "*ntsb_csv*".
2. Head over to the NTSB and get the  [CSV data](http://app.ntsb.gov/aviationquery/Download.ashx?type=csv).
3. Point your file input monitor at your data folder; select "*ntsb_csv*" as the sourcetype.
4. Install the app.

# Install to local Splunk Enterprise
1. Navigate to $SPLUNK_HOME/etc/apps
2. Execute: `git clone https://github.com/csyvenky/all_ntsb.git`
3. Restart Splunk: via Splunk Web or any other way you know how. Alternatively, you can use the Ansible Playbook 
*restart_splunk.yml* - a quick and dirty restart tool for splunkd.

---

# Create new app in Splunk Cloud
1. If you don't have an account, open a [free trial](https://www.splunk.com/getsplunk/cloud_trial).
2. Browse to your instance then: Apps | Manage Apps | Create App
Name: "*ntsb_csv*"
Folder: "*ntsb_csv*"
Version: "*1.0*"
Visible: "*Yes*"
Description: "*All NTSB Aviation Accident Database
Template: "*barebones*"

# Create new Index
1. Access your instance then: Settings | Indexes | New Index
Name: "*ntsb_csv*"
Size: "*35 MB*"
Retention: "*14,600 days*" (40 years)

# Populate the Index

## Get the raw data
1. Head over to the NTSB and download the [CSV data](http://app.ntsb.gov/aviationquery/Download.ashx?type=csv). Save a copy to your local workstation.

## Pretty it up
1. After downloading the data from NTSB.
2. Import into Excel | Data | From Text/CSV.
3. Delimit on the "*|*" char.
4. Create four new columns before the "*Country*" column.
5. Split the "*Location*" field to create new columns: "*State*", "*Location2*", "*Location3*", "*last/blank*" fields from the  (Text to Columns feature while delimiting on "*,*").
6. Delete the "last/blank" field (unused) field, it is likely still called "*Column4*".
7. Save as AviationData.csv and move the data file to a location where your Splunk's data file input configuration can import it. That's it for Excel.

## Create "*ntsb_csv*" Source Type
1. Access your instance then: Settings | Source Types | New Source Type
Name: "*ntsb_csv*"
Description: "*Process CSV for NTSB Aviation Accident Database*"
Destination App: The app created above; "*ntsb_csv*"
Indexed Extractions: "*csv*"
Timestamp: Advanced | Format: "*%Y-%m-%d*" | Timestamp Field: "*Event Date*"
Delimit on the "*,*" char.

## (optionally) Purge previous imports of same dataset
This step may be required if you have imported this same dataset another time - with the current configuration Splunk will happily import dupicate events. As the entire dataset is in each download we can safely import the entire dataset each time.

## Injest data into Splunk
1. Access the [file upload dialog](https://your_instance.cloud.splunk.com/en-GB/manager/launcher/adddata/selectsource?input_mode=0).
2. Upload the CSV (alternatively you could use a Splunk Forwarder to push the content of this file). Next.
3. Select "*ntsb_csv*" as source type. The preview of the data on the right hand side should be accurate with no warnings. Next.
4. Input settings | Host can be left as-is.
5. Input settings | Index should be set to "*ntsb_csv*". Review.
6. Submit.

## Validate Data Import
1. Access the [app homepage](https://your_instance.cloud.splunk.com/en-GB/app/all_ntsb/search).
2. Run the following search: "*index=ntsb_csv sourcetype=ntsb_csv*". In my instance I have > 82,000 events.
3. After all that work cleaning up the data, we should make sure that state was parsed correctly. Run the following search: "*index=ntsb_csv sourcetype=ntsb_csv | stats count by State | sort - count*"

# Create the Dashboard UI
1. Access the [Dashboards page](https://your_instance.cloud.splunk.com/en-GB/app/all_ntsb/dashboards) | Create New Dashboard.
Title: "*All NTSB Dashboard*"
Permissions: "*Share in App*"
2. Immediately edit the Dashboard in "*Source*" mode.
3. Copy and paste the code from the [github repo](https://raw.githubusercontent.com/csyvenky/all_ntsb/master/local/data/ui/views/ntsb.xml). Save.

# Add OurAirports.com Lookup Data
1. [Download the data file](https://github.com/csyvenky/all_ntsb/blob/master/lookups/ourairports_data.csv). Upload it via the control in Splunk Web.
2. Access the [Lookups page](https://your_instance.cloud.splunk.com/en-GB/manager/search/lookups). Add new.
Destination App: "*all_ntsb*"
Destnation FileName: "*ourairports_data.csv*"

# Add Lookup Definition
1. Access the [Lookups page](https://your_instance.cloud.splunk.com/en-GB/manager/all_ntsb/lookups). Add new.
Destination App: "*all_ntsb*"
Name: "*OurAirports.com*"
Lookup File: "*ourairports_data.csv*"

## Validate the Lookup
1. Access the [app homepage](https://your_instance.cloud.splunk.com/en-GB/app/all_ntsb/search).
2. Run the following search: "*| inputlookup ourairports_data.csv*".

---

This project includes some utility Ansible Playbooks for the Management of the app and data.

Playbooks:
1. *get_date_file.yml* - this script downloads the main datafile from the NTSB, there are two data formats (CSV and XML). The CSV file is a much smaller payload.
    ```
    ansible-playbook -i hosts get_data_file.yml --extra-vars "format=[csv|xml]" --ask-pass
    ```
2. *wipe_splunk_index.yml* - since the NTSB provides the data sets via an all-in-one file, we need a utitlity to wipe the index to prevent importing duplicate data into the index.
    ```
    ansible-playbook -i hosts wipe_splunk_index.yml --ask-pass
    ```
3. *restart_splunk.yml* - a quick and dirty restart tool for splunkd.
    ```
    ansible-playbook -i hosts restart_splunk.yml --ask-pass
    ```