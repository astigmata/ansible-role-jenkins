# Jenkins Ansible Role

Ansible role for deploying and configuring Jenkins CI/CD server with security best practices.

## Features

- ✅ Multi-OS support (Debian/Ubuntu, RHEL/CentOS)
- ✅ Idempotent operations
- ✅ Security hardening
- ✅ Plugin management via jenkins-cli
- ✅ Configurable authentication
- ✅ ansible-lint compliant

## Requirements

- Ansible >= 2.12
- Target system with systemd
- Internet access for package/plugin downloads

## Role Variables

### Core Configuration

```yaml
# Admin user credentials (use vault for password)
jenkins_admin_user: admin
jenkins_admin_password: "{{ vault_jenkins_admin_password }}"

# Service configuration  
jenkins_port: 8080
jenkins_host: "{{ ansible_default_ipv4.address }}"
jenkins_url: "http://{{ jenkins_host }}:{{ jenkins_port }}"

# Java options
jenkins_java_options: "-Djava.awt.headless=true -Xmx2g"
```

## Plugin Management

The role installs plugins using jenkins-cli for maximum compatibility:

```yaml
jenkins_plugins:
  - git
  - workflow-aggregator
  - pipeline-stage-view
  - credentials-binding
  - timestamper
  - ws-cleanup

jenkins_plugins_timeout: 300
```

**Plugin Installation Process:**
1. Downloads jenkins-cli.jar from Jenkins instance
2. Lists currently installed plugins
3. Installs missing plugins with dependencies
4. Restarts Jenkins if plugins were installed
5. Cleans up CLI jar file

**Note:** Plugin installation requires Jenkins restart, handled automatically by the role.

### Security Settings

```yaml
jenkins_security_realm: "HudsonPrivateSecurityRealm"
jenkins_authorization_strategy: "FullControlOnceLoggedInAuthorizationStrategy"
jenkins_anonymous_access: false
```

## Dependencies

None

## Example Playbook

### Basic Usage

```yaml
- hosts: jenkins-servers
  become: true
  vars:
    vault_jenkins_admin_password: "MySecurePassword123!"
  roles:
    - ansible-role-jenkins
```

### Advanced Configuration

```yaml
- hosts: jenkins-servers
  become: true
  vars:
    jenkins_port: 9090
    jenkins_java_options: "-Xmx4g -Djava.awt.headless=true"
    jenkins_plugins:
      - git
      - docker-plugin
      - kubernetes
      - blueocean
    vault_jenkins_admin_password: "{{ vault('jenkins_admin_password') }}"
  roles:
    - ansible-role-jenkins
```

## Security Considerations

### Password Management
Always use Ansible Vault for passwords:

```bash
ansible-vault create group_vars/all/vault.yml
```

```yaml
# vault.yml
vault_jenkins_admin_password: "YourSecurePassword123!"
```

### Firewall Configuration
Configure firewall rules for Jenkins port:

```yaml
# Additional task in playbook
- name: Configure firewall for Jenkins
  ansible.posix.firewalld:
    port: "{{ jenkins_port }}/tcp"
    permanent: true
    state: enabled
    immediate: true
```

## Testing

Test the role with molecule:

```bash
pip install molecule[docker]
molecule test
```

## Troubleshooting

### Common Issues

**Jenkins not starting:**
- Check Java installation: `java -version`
- Verify Jenkins logs: `journalctl -u jenkins -f`
- Check disk space: `df -h /var/lib/jenkins`

**Plugin installation fails:**
- Verify internet connectivity
- Check Java installation: `java -version`
- Test CLI manually: `java -jar /tmp/jenkins-cli.jar -s http://localhost:8080 -auth admin:password help`
- Increase `jenkins_plugins_timeout` value

**Authentication issues:**
- Verify admin user creation in logs
- Check `/var/lib/jenkins/users/{{ jenkins_admin_user }}/config.xml`
- Reset admin password manually if needed

### Health Checks

```bash
# Service status
systemctl status jenkins

# Web interface
curl -I http://localhost:8080/login

# CLI test
java -jar /tmp/jenkins-cli.jar -s http://localhost:8080 -auth admin:password who-am-i

# Plugin list
java -jar /tmp/jenkins-cli.jar -s http://localhost:8080 -auth admin:password list-plugins
```

## License

MIT

## Author Information

Created for healthcare DevOps environments with focus on security and compliance.