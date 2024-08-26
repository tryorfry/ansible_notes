# ansible_notes
ansible_training_notes


Ansible Notes from basic traning:
=============

Inventory
=========
  Inventory files inside the inventory directory
  ----------------------------------------------
    File Format: INI/YAML

    In INI inventory
      Can use range generator operators:
        server[1:5]   # this will generate server1 server2 server3 server4 server5
        host[c:e]     # this will generate hostc hostd hoste
      Create group of groups by using :children keyword
        [group_a]
        host-1
        host-2

        [group_b]
        host-3

        [big_group:children]
        group_a
        group_b

      Create group vars by using :vars keyword
        host-1    my_value1=100
        host-2    another_data="something long"

        [group_a:vars]
        value_x = 100
        message = "good morning"

    Use a custom name(alias) to refer to an existing host in your dc
      mr.troublemaker   ansible_host=server3.example.com

Configuration File
==================
  Location and Precedence
  -----------------------
    0. CMD Line (-i) option         # CMD Line options are always the highest precedence
    1. $ANSIBLE_CONFIG              # Environment Variable
    2. ./ansible.cfg                # current directory
    3. ~/.ansible.cfg               # hidden file name in home directory
    4. /etc/ansible/ansible.cfg     # Default

  Sample Configuration (INI Format)
  ---------------------------------
      ansible.cfg
      -----------
      [defaults]
      inventory   = INVENTORY_PATH
      roles_path  = ROLES_PATH
      log_path    = LOG_FILE
      remote_user = USER
      ask_pass    = true|False
      forks       = 5|NUM_OF_HOSTS                    # number of hosts to run per play
      transport   = Smart|winrm|network_cli           # win = winrm; ntdev = network_cli
      # Setting Collections Path
      collections_path   = COLLECTIONS_PATH
    # Extras
      gathering           = smart|Implicit|explicit   # "ansible-config list" for help
      host_key_checking   = True|false
      retry_files_enabled = True|false

    # Escalate privilege
      [privilege_escalation]
      become              = true|False
      become_user         = USER
      become_method       = su|Sudo|enable|runas|psexec # *nix = su,sudo; win = runas,psexec; ntdev = enable
      become_ask_pass     = true|False

      [inventory]
      enable_plugins=host_list, script, auto, yaml, ini, toml

      [ssh_connection]
      ssh_args = -o ControlMaster=auto -o ControlPersist=60s
      pipelining = true               # REQUIRES requiretty sudo option

Command Line Syntax
===================
  Documentation
  -------------
    ansible-doc [-t PLUGIN] -l|MODULE_NAME
      # ansible-doc -t connection -l
      # ansible-doc -t connection local
      # ansible-doc -t callback -l
      # ansible-doc -t become -l        # become_method settings
      # ansible-doc -l
      # ansible-doc command

  Config
  ------
    ansible-config list

  Ad-Hoc
  ------
    ansible --version|--help
    ansible [-i INVENTORY] [--list-hosts] HOST-PATTERN
    ansible HOST-PATTERN [-m MODULE] {-a 'ARGS...'} \
      [-i INVENTORY] [-u REMOTE-USER] [-k] [-f FORKS] [-v[v[v[v]]]] \
      [-b] [--become-method su|sudo|enable|runas] [--become-user USER] [-K]

      # ansible -i myinventory -m ios_command -a "commands='show version'" cs01
      # ansible webservers -a 'hostname'

  Playbook
  --------
    ansible-playbook PLAYBOOK [-e VAR=VALUE]... \
      [-i INVENTORY] [-u REMOTE-USER] [-k] \
      [-b] [--become-method su|sudo|enable|runas] [--become-user USER] [-K] \
      [-f FORKS] [--syntax-check] [-C] [--diff] [-v[v[v[v]]]] \
      [--list-tasks] [--[skip-]tags TAGS,..] \
      [--ask-vault-pass|{--vault-password-file PASSWORD_FILE}]

  Common options for ansible and ansible-playbook commands
  --------------------------------------------------------
    CMD Line options        ansible.cfg
    ----------------        -----------
                            # [defaults] section
    -i INVENTORY            inventory_path = PATH
    -u REMOTE_USER          remote_user = REMOTE_USER
    -k                      ask_pass = True/False
    -f FORKS                forks = FORKS
    -c CONNECTION_TYPE      transport = Smart|ssh|paramiko|local|winrm|network_cli

                            # [privilege_escalation] section
    -b                      become = True/False
    --become-user USER      become_user = USER
    --become_method METHOD  become_method = METHOD
    -K                      become_ask_pass = True/False

  Inventory
  ---------
    ansible-inventory [-i INVENTORY] [--graph|--list] [-y]

      # ansible-inventory --graph
      # ansible-inventory --list -y

  Vault
  -----
    ansible-vault create|edit|encrypt|decrypt|view|rekey \
      [--vault-password-file PASSWORD_FILE] FILENAME

  Galaxy
  ------
    ansible-galaxy collection|role init|install ROLE_NAME

      # ansible-galaxy collection init mycollection
      # ansible-galaxy role init myrole

  Navigator
  ---------
      ansible-navigator [--help]
      ansible-navigator run [--hp]
      ansible-navigator doc [-t PLUGIN] -l
      ansible-navigator doc MODULE -m stdout
      ansible-navigator run [--ee True|false] [--eei IMAGE] [--eev LOCAL_PATH:CONT_PATH] [-i INVENTORY] [--pp PULL_POLICY] PLAYBOOK
      ansible-navigator inventory -i INVENTORY [--list|--graph]
      
      HELP:
      ansible-navigator [--help]
      ansible-navigator doc [--hd]
      ansible-navigator config [--hc]
      ansible-navigator inventory [--hi]
      ansible-navigator run [--hp|--help]

YAML
====

  ---   # start of file - optional
  ...   # end of file - optional

  LIST always starts with a dash (followed by space!!!)
    - hosta
    - hostb
    - groupa

  DICTIONARY is a key-value pair separated by colon (followed by space!!!)
    item: abc
    msg: Hello World
    value1: 12345

  multi line can be written as
    address: |
      1, Block A,
      ABC Avenue,
      1234 FairyLand.

  single line can be written as
    long_line: >
      This is a very very very long single
      line message that is written with
      multiple repeats, repeating over
      and over and over and over and over.

  Abbrevation of dictionary and list
  LONG                    SHORT
  ----                    -----
  user:                   user: {name: John, age: 12}
    name: John
    age: 12

  store_item:             store_item: ['banana', 'apple', 'cherry']
    - banana
    - apple
    - cherry

  user_list:              user_list:
    - name: one             - {name: one, age: 1}
      age: 1                - {name: two, age: 2}
    - name: two
      age: 2


  instead of single very long line
    when: ansible_facts['distribution'] == "Microsoft Windows Server 2016 Datacenter" or ansible_facts['distribution'] == "Microsoft Windows Server 2012 Datacenter"

  you can make use of > and write it this way
    when: >
      ansible_facts['distribution'] == "Microsoft Windows Server 2016 Datacenter" or 
      ansible_facts['distribution'] == "Microsoft Windows Server 2012 Datacenter"

  Ref: https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html


Playbook Syntax
===============
  ---                                     # start of YAML (optional)
  - name: PLAY_NAME
    hosts: HOST_LIST
    vars: VAR_DICTS
    vars_files: FILE_LIST                 # higher priority over the vars
    order: inventory|reverse_inventory|sorted|reverse_sorted|shuffle
    become: true|false
    gather_facts: True|false              # when dealing with network devices set to False
    connection: PLUGIN                    # overrides transport in defaults section of ansible.cfg
    collections: COLLECTION_LIST          # use FQCN instead, if possible
    ignore_errors: true|False             # dangerous. Try not to use this if possible.
    force_handlers: False|true
    serial: NUMBER[%]                     # number/percentage of host to run per play
    max_fail_percentage: PERCENTAGE       # acceptable failure percentage per play run
    pre_tasks: TASK_LIST
    roles:                                # can be written as - {role: ROLE, var: VALUE, ...}
      - role: ROLE_NAME                  ╶┬╴ ROLE
        VAR_DICTS                         │
        tags: TAG_LIST                    │
        become: true|false               ╶╯
    tasks:
      - name: TASK_NAME                  ╶┬╴ TASK
        vars: VAR_DICTS                   │
        MODULE_NAME: MOD_OPT_DICTS        │
        become: true|false                │
        register: VAR_NAME                │
        ignore_errors: true|False         │
        run_once: true|False              │
        loop: CONDITION_LIST              │
        when: CONDITION_LIST              │   # before task execution
        changed_when: CONDITION_LIST      │   # after task execution
        failed_when: CONDITION_LIST       │   # after task execution
        delegate_to: INVENTORY_HOSTNAME   │
        delegate_facts: true|False        │
        tags: TAG_LIST                    │
        until: CONDITION_LIST             │
        retries: NUM                      │
        delay: DELAY_IN_SECONDS           │
        notify: HANDLER_LIST             ╶╯
        listen: HANDLER_NAME              # ONLY used under handlers section

      - block: TASK_LIST                 ╶┬╴ BLOCK
        rescue: TASK_LIST                 │
        always: TASK_LIST                 │
        become: true|false                │
        when: CONDITION_LIST             ╶╯
    post_tasks: TASK_LIST
    handlers: TASK_LIST

  - name: Import Playbook
    import_playbook: PLAYBOOK
  ...

  Conditional Symbols
  ===================
   ==           equal to
   !=           not equal to
   >            greater than
   <            less than
   >=           greater than or equal to 
   <=           less than or equal to
   is defined   variable is defined
   is not defined   variable is not defined
   in LIST          variable/value is in a LIST

  HANDLERS
  ========
    RULES
    -----
      Handlers will only execute:
        1. The task that notified the handler has a status of "changed"
        2. In the order defined, and not the notified order.
        3. Once regardless of how many times it was notified.
        4. After ALL TASKS have completed for the PLAY.
              EXCEPTION - force_handlers: true
              EXCEPTION - calling to meta module with flush_handlers argument as one of the tasks.
                - name: running notified handlers
                  meta: flush_handlers
        5. In their own namespace. (You can't notify a playbooks handler from inside a role and vice versa)

      Handlers will execute based on the above rules each time for pre_tasks, tasks and post_tasks.
        1. pre_tasks
        2.   handlers
        3. roles
        4. tasks
        5.   handlers
        6. post_tasks
        7.   handlers

Variables
=========
  SCOPE
  -----
    3 Types:
      1. Global - Variables can be used anywhere in Playbook
      2. Play   - Variables can only be used in Play/Task where they were declared
      3. Host   - Variables can be used in ANY Playbook
                  Also known as inventory variables
                  Only available in hosts where they were declared

              Precedence                          Validity
              ----------                          --------
    1. Global 1. -e CMD LINE OPTION               PLAYBOOK # cmd line or ansible.cfg config
    2. Play                                       PLAYBOOK # play level
              2. INCLUDE_VARS MODULE              PLAY - FROM INCLUDE TASK ONWARDS
              3. TASK VARS                        Task ONLY
              4. PLAY VARS_FILES                  PLAY
              5. PLAY VARS                        PLAY
    3. Host                                       # inventory variables
              6. PLAYBOOK HOST_VARS               ╶┬╴Playbook
              7. INVENTORY HOST_VARS               │
              8. INVENTORY FILE HOST variables     │
              9. PLAYBOOK GROUP_VARS               │
              10. INVENTORY GROUP_VARS             │
              11. PLAYBOOK GROUP_VARS/all          │
              12. INVENTORY GROUP_VARS/all         │
              13. INVENTORY FILE GROUP VARIABLES  ╶╯


  Variables Precedence
  --------------------
    PROJECT/
      ├╴ansible.cfg                           # INI Syntax
      ├╴group_vars/
      │ ├╴all                   6
      │ └╴GROUP_NAME            4
      ├╴host_vars/
      │ └╴INVENTORY_HOSTNAME    1
      ├╴inventory/
      │ ├╴group_vars/
      │ │ ├╴all                 7
      │ │ └╴GROUP_NAME          5
      │ ├╴host_vars/
      │ │ └╴INVENTORY_HOSTNAME  2
      │ └╴INVENTORY_FILE        3h 8g         # INI Syntax
      └╴playbook.yaml

  Internal Variables
  ------------------
    ansible_connection: local|Smart|ssh|paramiko|winrm
  # windows
    ansible_winrm_transport: basic/credssp
    ansible_winrm_server_cert_validation: ignore 
    ansible_port: 5986
  # network - usually in group_vars
    ansible_connection: network_cli|netconf         # ansible.netcommon.network_cli
    ansible_network_os: ios|nxos|eos|enos|vyos...   # refer to Extras
    ansible_become_password: PASSWORD
  # common
    ansible_user: USERNAME
    ansible_password: PASSWORD

    ansible_host: <IPADDR|FQDN>
    ansible_become: true|false
    ansible_become_user: <USER>
    ansible_become_method: su|Sudo|psexec|runas|enable
    ansible_python_interpreter: auto|<LOCATION>

    When identifying current managed hosts:
      inventory_hostname
      ansible_host
      ansible_facts['hostname']   # short hostname
      ansible_facts['fqdn']       # fully qualified domain name
      ansible_play_hosts          # remainding hosts that haven't failed. used internally.

    Extras:
      become     - https://docs.ansible.com/ansible/latest/user_guide/become.html
      network_os - https://docs.ansible.com/ansible/latest/network/user_guide/platform_index.html

  Magic Variables
  ===============
    inventory_hostname      # inventory host name
    groups                  # List of all inventory groups and their members
    group_names             # List of all groups that the current inventory host belongs

    Eg.
      Assuming you're inside servera now, to refer to it's own groups magic var:
        hostvars.servera.groups OR
        hostvars['servera']['groups'] OR
        groups
      Structure of hostvars is as below:

      hostvars:
        servera:
          inventory_hostname: servera # each host has it's magic var
          
          group_names:
            - webservers
            - nodes
          groups:
            all:
              - monkey
              - servera
              - serverb
            nodes:
              - servera
              - serverb
            webservers:
              - servera
            dbservers:
              - serverb
            ungrouped:
              - monkey
          ansible_facts: {}
        serverb:
          inventory_hostname: serverb
          ...

Extras:
  forks  -> number of ssh sessions to spawn per TASK
         -> set using the -f option or "forks" setting in defaults section of ansible.cfg
  serial -> execute whole PLAY for SERIAL number of nodes
         -> set in playbook at play level

  Config Settings   - https://docs.ansible.com/ansible/latest/reference_appendices/config.html
  Playbook Keywords - https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html
  Playbook Filters  - https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html
  Jinja2            - https://jinja.palletsprojects.com/en/3.1.x/templates

Jinja2
======
  Delimeters:
    {% ... %} - statements
    {{ ... }} - expressions to generate as output
    {# ... #} - comments not included in output

  Conditional Statements:
    {% if CONDITION %}
    ...
    {% else %}
    ...
    {% endif %}

  Loops:
    {% for VAR in LIST %}
    {% if CONDITION %}
    ...
    {% endif %}
    {% endfor %}

    {% for VAR in LIST [if CONDITION] %}
    ...
    {% endfor %}
