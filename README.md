An experiment with ansible roles and variable scope

Deals with topic mentioned in this thread:

https://groups.google.com/forum/#!topic/Ansible-project/yrnsx2Mw6rc

This defines a playbook with 3 roles:

- foo
- bar
- dog

Each role does an include to pick up a task in another role called
"pythonapp" - this task simply prints out the value of a variable called
`sm_app_role`.

The roles are supposed to define the `sm_app_role` variable in a
`vars/main.yml` file...

But we neglect to do this in the `bar` role (simulating programmer
error)...

Since I thought that variables are scoped to roles, I expected this output:

    TASK: [bar | print value of sm_app_role] **************************************
    ok: [localhost] => {
        "sm_app_role": "{{ sm_app_role }}"
    }

But instead I get this:

    TASK: [bar | print value of sm_app_role] **************************************
    ok: [localhost] => {
        "sm_app_role": "dog"
    }

The latter seems to suggest that in at least some cases there is leakage of
variables from one role to another. Why?

Output:

```
$ ansible-playbook -vvv -i hosts playbook.yml

PLAY [localhost] **************************************************************

GATHERING FACTS ***************************************************************
<localhost> REMOTE_MODULE setup
<localhost> EXEC ['/bin/sh', '-c', 'mkdir -p $HOME/.ansible/tmp/ansible-tmp-1404315384.66-20719838060817 && chmod a+rx $HOME/.ansible/tmp/ansible-tmp-1404315384.66-20719838060817 && echo $HOME/.ansible/tmp/ansible-tmp-1404315384.66-20719838060817']
<localhost> PUT /var/folders/gw/w0clrs515zx9x_55zgtpv4mm0000gp/T/tmpBZ1TEE TO /Users/marca/.ansible/tmp/ansible-tmp-1404315384.66-20719838060817/setup
<localhost> EXEC ['/bin/sh', '-c', u'LC_CTYPE=en_US.UTF-8 LANG=en_US.UTF-8 /usr/bin/python /Users/marca/.ansible/tmp/ansible-tmp-1404315384.66-20719838060817/setup; rm -rf /Users/marca/.ansible/tmp/ansible-tmp-1404315384.66-20719838060817/ >/dev/null 2>&1']
ok: [localhost]

TASK: [foo | print value of sm_app_role] **************************************
ok: [localhost] => {
    "sm_app_role": "foo"
}

TASK: [bar | print value of sm_app_role] **************************************
ok: [localhost] => {
    "sm_app_role": "dog"
}

TASK: [dog | print value of sm_app_role] **************************************
ok: [localhost] => {
    "sm_app_role": "dog"
}

PLAY RECAP ********************************************************************
localhost                  : ok=4    changed=0    unreachable=0    failed=0


$ ansible-playbook --version
ansible-playbook 1.6.5
```
