# Ansible Playbook: Create libvirt Storage Pool

This Ansible playbook is designed to create a libvirt storage pool and ensure its activation and proper configuration.

## Prerequisites

- The `community.libvirt` collection should be installed using `ansible-galaxy collection install community.libvirt`.

## Playbook Explanation

### Variables
- `libvirt_folder`: The directory path for libvirt.
- `image_pool_dir`: Directory where the storage pool will be created.
- `image_pool_name`: Name of the storage pool.
- `image_pool_type`: Type of storage pool, in this case, it's "dir".
- `libvirt_uri`: URI for libvirt connection, default is "qemu:///session".

#### Variables Defaults in playbook
```yaml
# Default variables in the create_virt_pool.yml
vars:
  libvirt_folder: "/var/lib/libvirt/"
  image_pool_dir: "/opt/pools/homelab/"
  image_pool_name: "ansible_pool"
  image_pool_type: "dir"
```

### Play Tasks

1. **Gather Facts on Existing virsh Pool:**
   - Collect information about existing virsh pools using the `community.libvirt.virt_pool` module and store it in `ansible_libvirt_pools`.

2. **Add a Storage Pool:**
   - Define a storage pool if it doesn't already exist using the `community.libvirt.virt_pool` module with the "define" command. The pool's configuration is taken from a template XML file.

3. **Create Image Pool Directory:**
   - Ensure the directory for the image pool exists with specific ownership and permissions using the `ansible.builtin.file` module.

4. **Build and Start the Storage Pool:**
   - If the pool is not active, build and start the storage pool using the `community.libvirt.virt_pool` module with the "build" and "create" commands respectively.

5. **Autostart the Storage Pool:**
   - Enable autostart for the storage pool using the `community.libvirt.virt_pool` module.

6. **Ensure the Pool is Active:**
   - Ensure that the pool is in an "active" state using the `community.libvirt.virt_pool` module.

## Running the Playbook

Execute this playbook using the following command:

```bash
ansible-playbook create_virt_pool.yml
```

## Documentation for the `pool.xml.j2` template:

```xml
<!-- pool.xml.j2 -->

This XML template is used to generate the configuration for defining a libvirt storage pool.

### Template Explanation

- `<pool>`: Defines the type of pool with attributes and elements.
  - `type`: Represents the type of the storage pool, derived from the `image_pool_type` variable.
  
- `<name>`: Specifies the name of the storage pool, taken from the `image_pool_name` variable.
  
- `<target>`: Contains information about the storage pool's target path.
  - `<path>`: Indicates the directory path where the storage pool will be created. This value is derived from the `image_pool_dir` variable.

### Template 
This template is utilized within the Ansible playbook to dynamically generate the XML configuration for defining the libvirt storage pool.

```jinja2
<pool type='{{ image_pool_type }}'>
    <name>{{ image_pool_name }}</name>
    <target>
        <path>{{ image_pool_dir }}</path>
    </target>
</pool>
```
### Template eaxample how it is used in the file create_virt_pool.yaml
```yaml
    - name: add a storage pool for storing libvirt images if Pool doesn't already exists
      community.libvirt.virt_pool:
        command: define
        # looks like setting name here is a redundant, the name is anyways taken from the template xml file, but should set it to make virt_pool module happy.
        name: "{{ image_pool_name }}"
        xml: '{{ lookup("template", "pool/pool.xml.j2") }}'
        uri: "{{ libvirt_uri }}"
      when: image_pool_name not in ansible_libvirt_pools
```
Replace variables `{{ image_pool_type }}`, `{{ image_pool_name }}`, and `{{ image_pool_dir }}` with the desired values when using this template.
```

