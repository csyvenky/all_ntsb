all_ntsb Splunk App
======

*This is a work in progress.*

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