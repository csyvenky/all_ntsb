# all_ntsb Splunk App
Splunk app to process NTSB Safety Data

*This is a work in progress.*

# Setup
1. Create an index on your Splunk instance, I called mine "*ntsb_csv*".
2. Head over to the NTSB and get the  [CSV data](http://app.ntsb.gov/aviationquery/Download.ashx?type=csv).
3. Point your file input monitor at your data folder; select "*ntsb_csv*" as the sourcetype.
4. Install the app.

# Install
1. Navigate to $SPLUNK_HOME/etc/apps
2. Execute: `git clone https://github.com/csyvenky/all_ntsb.git`
3. Restart Splunk: via Splunk Web or any other way you know how. Alternatively, you can use the Ansible Playbook 
*restart_splunk.yml* - a quick and dirty restart tool for splunkd.

This includes some utility Ansible Playbooks for the Management of the app and data.

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