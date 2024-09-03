# Log Management with Ansible

## Overview

This project provides an Ansible playbook designed to manage and organize log files on a Windows server. The playbook searches for log files generated on the current day and the previous day within a specified directory structure and copies them to a dedicated folder for easy access and analysis.

## Directory Structure Example

```plaintext
F:\Logs
  ├── cashEx
  │   ├── 20240803-log.xml
  │   ├── 20240802-log.xml
  │   └── ...
  ├── ASSur
  │   ├── 20240803-log.xml
  │   ├── 20240802-log.xml
  │   └── ...
  ├── Filla
  │   ├── 20240802-log.xml
  │   ├── 20240801-log.xml
  │   └── ...
  └── Moni
      ├── 20240801-log.xml
      ├── 20240731-log.xml
      └── ...
```
After running the playbook, the logs from today and yesterday will be copied to the following structure:
```plaintext
F:\Logs_Today_Yesterday
  ├── Logs_Today
  │   ├── cashEx-20240803-log.xml
  │   ├── ASSur-20240803-log.xml
  │   └── ...
  └── Logs_Yesterday
      ├── cashEx-20240802-log.xml
      ├── ASSur-20240802-log.xml
      ├── Filla-20240802-log.xml
      └── ...
```

## Ansible Playbook

To create an Ansible playbook that searches for today's and yesterday's logs in the subfolders of **F:\Logs**, and then copies them into a new folder **Logs_Today_Yesterday**, here is an example playbook you can use:

```yaml
---
- name: Collect today's and yesterday's logs
  hosts: win
  tasks:
    - name: Get today's date in yyyymmdd format
      ansible.builtin.win_command: powershell -Command "(Get-Date).ToString('yyyyMMdd')"
      register: today_date
      changed_when: false

    - name: Get yesterday's date in yyyymmdd format
      ansible.builtin.win_command: powershell -Command "(Get-Date).AddDays(-1).ToString('yyyyMMdd')"
      register: yesterday_date
      changed_when: false

    - name: Create destination folder for today's and yesterday's logs
      ansible.builtin.win_file:
        path: F:\Logs_Today_Yesterday
        state: directory

    - name: Find today's logs in F:\Logs
      ansible.builtin.win_find:
        paths: F:\Logs
        recurse: yes
        patterns: "*{{ today_date.stdout }}*"
      register: todays_logs

    - name: Find yesterday's logs in F:\Logs
      ansible.builtin.win_find:
        paths: F:\Logs
        recurse: yes
        patterns: "*{{ yesterday_date.stdout }}*"
      register: yesterdays_logs

    - name: Copy today's logs to F:\Logs_Today_Yesterday with folder name prefix
      ansible.builtin.win_copy:
        src: "{{ item.path }}"
        dest: "F:\Logs_Today_Yesterday\{{ item.path | dirname | basename }}-{{ item.path | basename }}"
      with_items: "{{ todays_logs.files }}"

    - name: Copy yesterday's logs to F:\Logs_Today_Yesterday with folder name prefix
      ansible.builtin.win_copy:
        src: "{{ item.path }}"
        dest: "F:\Logs_Today_Yesterday\{{ item.path | dirname | basename }}-{{ item.path | basename }}"
      with_items: "{{ yesterdays_logs.files }}"

```

## Playbook Explanation
#### 1. Get today's and yesterday's dates:
The tasks Get today's date and Get yesterday's date use PowerShell commands to retrieve today's and yesterday's dates in the yyyymmdd format.
#### 2. Create destination folders:
This task creates the Logs_Today and Logs_Yesterday folders under F:\Logs_Today_Yesterday if they do not already exist.
#### 3. Find today's and yesterday's logs:
The tasks Find today's logs and Find yesterday's logs use the win_find module to search for files matching today's and yesterday's dates in the subfolders of F:\Logs.
#### 4. Copy logs:
The tasks Copy today's logs and Copy yesterday's logs copy the found files to the destination folders. The files are renamed to include the date in their names.

## Step-by-Step Explanation with Scenario
#### Step 1: Retrieve Today's Date
```yml
- name: Get today's date in yyyymmdd format
  ansible.builtin.win_command: powershell -Command "(Get-Date).ToString('yyyyMMdd')"
  register: today_date
```
Assume today is August 3, 2024. This task stores 20240803 in the variable today_date.

#### Step 2: Create a Folder for Today's Logs
```yml
- nam: Create destination folder for today's logs
  ansible.builtin.win_file:
    path: F:\Logs_Today
    state: directory
```
This task creates the F:\Logs_Today folder where we will copy the found logs.

#### Step 3: Search for Today's Log Files
```yaml
- name: Find today's logs in F:\Logs
  ansible.builtin.win_find:
    paths: F:\Logs
    recurse: yes
    patterns: "*{{ today_date.stdout }}*"
  register: todays_logs
```
Here, Ansible searches all files in F:\Logs and its subfolders that contain 20240803 in their name, for example, 20240803-log.xml.

#### Step 4: Copy Today's Logs to the Logs_Today Folder
```yaml
- name: Copy today's logs to F:\Logs_Today with folder name prefix
  ansible.builtin.win_copy:
    src: "{{ item.path }}"
    dest: "F:\Logs_Today\{{ item.path | dirname | basename }}-{{ item.path | basename }}"
  with_items: "{{ todays_logs.files }}"
```
This final task copies the found files into F:\Logs_Today, keeping their original names.

