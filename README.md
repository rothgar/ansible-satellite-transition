# Ansible Satellite Transition
This playbook will move nodes registered in one Satellite host to another. *It will not install satellite/katello server for you.* Use this after you have set up a new Satellite server and want to move all your existing nodes from one server to another.

# Configure Playbook
Copy the hosts.template file and fill it out with information for your infrastructure. Add systems to [nodes] for hosts you want tasks to run on.

```[satellite]
satellite.example.com

[old_satellite]
satellite.example.com

[puppet_master]
satellite.example.com

[puppet_ca]
satellite.example.com

[nodes]
node1.example.com
node2.example.com
 
[6RedHatEnterpriseServer:vars]
activationkey='server,6epel'

[7RedHatEnterpriseServec:vars]
activationkey='workstation,6epel'
```
Use activation keys to register the hosts so make sure your activation keys are set up in satellite before running.

#Running

Enable the satellite settings create_new_host_when_facts_are_uploaded and create_new_host_when_report_is_uploaded to have hosts automatically created after puppet runs. You should also enable a default_location and default_organization in satellite. These settings are all under the puppet tab.

To run on all of your nodes (defined in hosts) make sure you update the activationkey variables (in hosts) and then use.

`ansible-playbook -i hosts satellite-playbook.yaml`

Add `-k` (ssh) or `-K` (sudo) if you need password prompts.

You can also run just the puppet registration tasks with

`ansible-playbook -i hosts satellite-playbook.yaml --tags puppet`

After the tasks complete you should have new unmanaged hosts in satellite. Edit the host and add any configuration you need (host groups, network, puppet). Unfortunately, I could not find a way to automate those steps yet. Your best bet is probably [hammer](https://github.com/theforeman/hammer-cli).

Once the hosts have been moved you may need to reinstall the katello-agent. Do that with `ansible all -i hosts -m yum -a "state=absent name=katello-agent"` and then `ansible all -i hosts -m yum -a "state=present name=katello-agent"`
