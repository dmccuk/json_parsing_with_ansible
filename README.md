# JSON parsing examples
This repo will grow overtime and give examples of parsing JSON to be able to get to the bit of information you want.

### JSON example
The json file I'll be using in this example.

````
[
  {
    "?xml": {
      "attributes": {
        "encoding": "UTF_8",
        "version": "1.0"
      }
    }
  },
  {
    "jdbc_data_source": [
      {
        "attributes": {
          "xmlns": "http://xmlns.oracle.com/weblogic/jdbc_data_source"
        }
      },
      {
        "name": "cwdsjndi"
      },
      {
        "jdbc_driver_params": [
          {
            "url": "jdbc:oracle:thin:@//myhost.myshop.com:1521/OLTT206"
          },
          {
            "driver_name": "oracle.jdbc.OracleDriver"
          },
          {
            "properties": {
              "property": [
                {
                  "name": "user7"
                },
                {
                  "value": "CAN_USER"
                }
              ]
            }
          },
          {
            "password_encrypted": "{AES}BcqmURyYoCkLvC5MmREXsfpRMO93KPIubqUAbb95+nE="
          }
        ]
      },
      {
        "jdbc_data_source_params": {
          "jndi_name": "cwds"
        }
      }
    ]
  },
  {
    "?xml": {
      "attributes": {
        "encoding": "UTF_8",
        "version": "1.0"
      }
    }
  },
  {
    "jdbc_data_source": [
      {
        "attributes": {
          "xmlns": "http://xmlns.oracle.com/weblogic/jdbc_data_source"
        }
      },
      {
        "name": "dsvelcw"
      },
      {
        "jdbc_driver_params": [
          {
            "url": "jdbc:oracle:thin:@myhost.myshop.com:1521:DB01"
          },
          {
            "driver_name": "oracle.jdbc.OracleDriver"
          },
          {
            "properties": {
              "property": [
                {
                  "name": "user"
                },
                {
                  "value": "WEB_USER"
                }
              ]
            }
          },
          {
            "password_encrypted": "{AES}WYImLDLo0j7S80cGMUDsE5M5QTnpffTPGyIzDuGG6WU="
          }
        ]
      },
      {
        "jdbc_data_source_params": {
          "jndi_name": "dsvelcw"
        }
      }
    ]
  }
]
````

## Example 1
In this example, I use debug to pull out the information:

````
---
- name: ReadJsonfile
  hosts: localhost
  tasks:
  - name: Display the JSON file content
    shell: "cat file.json"
    register: result

  - name: debug
    set_fact:
      result1: "{{ result.stdout | from_json }}"

  - name: debug
    debug:
      msg: "jdbc_data_source name is {{ result1[1].jdbc_data_source[3].jdbc_data_source_params.jndi_name }} has username {{ result1[1].jdbc_data_source[2].jdbc_driver_params[2].properties.property[1].value }} and jndi name {{ result1[1].jdbc_data_source[1].name }}"

  - name: debug
    debug:
      msg: "jdbc_data_source name is {{ result1[1].jdbc_data_source[3].jdbc_data_source_params.jndi_name }}"

  - name: debug
    debug:
      msg: "has username {{ result1[1].jdbc_data_source[2].jdbc_driver_params[2].properties.property[1].value }}"  
  - name: debug
    debug:
      msg: "and jndi name {{ result1[1].jdbc_data_source[1].name }}"
````

### Example 1 ansible output
This is what ansible will give you.

````
$ ansible-playbook one.yml

PLAY [ReadJsonfile] ***********************************************************************************************

TASK [Gathering Facts] ********************************************************************************************
ok: [localhost]

TASK [Display the JSON file content] ******************************************************************************
changed: [localhost]

TASK [debug] ******************************************************************************************************
ok: [localhost]

TASK [debug] ******************************************************************************************************
ok: [localhost] => {
    "msg": "jdbc_data_source name is cwds has username CAN_USER and jndi name cwdsjndi"
}

TASK [debug] ******************************************************************************************************
ok: [localhost] => {
    "msg": "jdbc_data_source name is cwds"
}

TASK [debug] ******************************************************************************************************
ok: [localhost] => {
    "msg": "has username CAN_USER"
}

TASK [debug] ******************************************************************************************************
ok: [localhost] => {
    "msg": "and jndi name cwdsjndi"
}

PLAY RECAP ********************************************************************************************************
localhost                  : ok=7    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 

````

## Example 2
In this example, I use ```set_fact:``` to pull register the variables and then use debug to display the variables. This example is the better one to use.

````
---
- name: ReadJsonfile
  hosts: localhost
  vars:
    input: "{{ lookup('file','file.json') | from_json }}"
    urls: "[?contains(xmlns, 'http')]"
  tasks:

  - name: Set our facts from the JSON file
    set_fact:
      sourcename: "{{ input[1].jdbc_data_source[3].jdbc_data_source_params.jndi_name }}"
      username: "{{ input[1].jdbc_data_source[2].jdbc_driver_params[2].properties.property[1].value }}"
      jndiname: "{{ input[1].jdbc_data_source[1].name }}"

  - name: debug
    debug:
      var: sourcename

  - name: debug
    debug:
      var: username

  - name: debug
    debug:
      var: jndiname

  - name: debug
    debug:
      msg: jdbc_data_source name is {{ sourcename }} has username {{ username }} and jndi name {{ jndiname }}
````

### Example 2 output
This is what the second playbook will give you.

````
$ ansible-playbook two.yml

PLAY [ReadJsonfile] ***********************************************************************************************

TASK [Gathering Facts] ********************************************************************************************
ok: [localhost]

TASK [Set our facts from the JSON file] ***************************************************************************
ok: [localhost]

TASK [debug] ******************************************************************************************************
ok: [localhost] => {
    "sourcename": "cwds"
}

TASK [debug] ******************************************************************************************************
ok: [localhost] => {
    "username": "CAN_USER"
}

TASK [debug] ******************************************************************************************************
ok: [localhost] => {
    "jndiname": "cwdsjndi"
}

TASK [debug] ******************************************************************************************************
ok: [localhost] => {
    "msg": "jdbc_data_source name is cwds has username CAN_USER and jndi name cwdsjndi"
}

PLAY RECAP ********************************************************************************************************
localhost                  : ok=6    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
````
