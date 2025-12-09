
| **Tool**         | **Best For**                             | **Primary Language** | **Notes**                                       |
| ---------------- | ---------------------------------------- | -------------------- | ----------------------------------------------- |
| **Serverspec**   | General server and config validation     | Ruby                 | Great for testing basic server configurations.  |
| **InSpec**       | Compliance, security checks              | Ruby                 | Ideal for security audits and compliance.       |
| **Testinfra**    | Lightweight testing with Python          | Python               | Good for Python-based projects or CI.           |
| **Molecule**     | Testing Ansible roles and playbooks      | Python/YAML          | Integrated with Ansible, good for CI pipelines. |
| **Goss**         | Fast, YAML-based testing for containers  | Go                   | Lightweight, YAML syntax, ideal for Docker.     |
| **RSpec-Puppet** | Testing Puppet configurations            | Ruby                 | Puppet-specific, for unit-testing manifests.    |
| **ChefSpec**     | Testing Chef cookbooks                   | Ruby                 | Great for Chef-managed environments.            |
| **Bats**         | Basic, shell-script-based server testing | Bash                 | Simple, fast, suitable for Unix environments.   |

#### Testinfra

Packages
`python3, python3-pip`

Unit test to test if local `sshd` service and running and enabled and port 22 is open

```
import unittest
import testinfra

class TestSSHdService(unittest.TestCase):

  def setUp(self):
    self.host = testinfra.get_host("local://")

  def test_sshd_service_running(self):
    sshd = self.host.service("sshd")
    self.assertTrue(sshd.is_running, "sshd service should be running")
    self.assertTrue(sshd.is_enabled, "sshd service should be enabled at boot")

  def test_port_22_open(self):
    port = self.host.socket("tcp://0.0.0.0:22")
    self.assertTrue(port.is_listening, "Port 22 should be open and listening")

if __name__ == "__main__":
  unittest.main()
```

