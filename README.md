# 4640-pod1

```BASH
ansible-playbook task1_create_user.yml
ansible-playbook task2_install_podman.yml
```

These 2 commands above will run the playbooks.

Playbook 1:
```YAML
---
  - hosts: all
    remote_user: root
    become: true

    tasks:
      - name: add a new user to the VMs named
        user:
          name: HAHA
          password: ''
          shell: /bin/bash
          create_home: true
          home: /home/HAHA

      - name: give the new user sudo in Ubuntu Distro
        ansible.builtin.shell: usermod -aG sudo HAHA
        when: ansible_distribution in ["Ubuntu"]

      - name: give the new user sudo (WHEEL) in Rocky Distro
        ansible.builtin.shell: usermod -aG wheel HAHA
        when: ansible_distribution in ["Rocky"]

      - name: create .ssh directory
        file:
          path: /home/HAHA/.ssh
          state: directory
          mode: 700

      - name: move .ssh to the users home directory
        become: true
        ansible.builtin.copy:
          remote_src: yes
          src: .ssh/authorized_keys
          dest: /home/HAHA/.ssh/authorized_keys
          owner: HAHA
          group: HAHA

      - name: remove ssh from root user
        become: true
        lineinfile:
          dest: /etc/ssh/sshd_config
          regexp: '^PermitRootLogin'
          line: "PermitRootLogin no"
          state: present

      - name: restart ssh
        become: true
        service:
          name: sshd
          state: restarted
```
Playbook 2:
```YAML
---
  - hosts: all
  remote_user: HAHA
  become: true

  tasks:
    - name: install podman
      package:
        name: "podman"
        state: present

    - name: pull an image
      containers.podman.podman_image:
        name: nginx
```
   
