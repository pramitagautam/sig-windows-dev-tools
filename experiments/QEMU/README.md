As mentioned in the issue https://github.com/kubernetes-sigs/sig-windows-dev-tools/issues/218, the current code base doesnt work on the arm based machines like Mac M1.
As a solution, I tried qemu plugin in vagrant. 
Below are the steps, that were done:-

```brew install --cask vagrant
brew install make gcc qemu
vagrant plugin install vagrant-qemu

echo 'security_driver = "none"' >> /usr/local/etc/libvirt/qemu.conf
echo "dynamic_ownership = 0" >> /usr/local/etc/libvirt/qemu.conf
echo "remember_owner = 0" >> /usr/local/etc/libvirt/qemu.conf

vagrant up --provider qemu
```

The above command was working for arm based boxes.
But when i use it with the boxes present in the source code that are amd64 based like (2004/ubuntu), 
VM was not coming up and giving the below error:-

```DEBUG ssh: == Net-SSH connection debug-level log START ==
DEBUG ssh: D, [2023-02-03T20:44:58.673978 #3375] DEBUG -- net.ssh.transport.session[135d8]: establishing connection to 127.0.0.1:50022
D, [2023-02-03T20:44:58.674381 #3375] DEBUG -- net.ssh.transport.session[135d8]: connection established
I, [2023-02-03T20:44:58.674406 #3375]  INFO -- net.ssh.transport.server_version[135ec]: negotiating protocol version
D, [2023-02-03T20:44:58.674413 #3375] DEBUG -- net.ssh.transport.server_version[135ec]: local is `SSH-2.0-Ruby/Net::SSH_7.0.1 x86_64-darwin19'

DEBUG ssh: == Net-SSH connection debug-level log END ==
 INFO ssh: SSH not ready: #<Vagrant::Errors::NetSSHException: An error occurred in the underlying SSH library that Vagrant uses.
The error message is shown below. In many cases, errors from this
library are caused by ssh-agent issues. Try disabling your SSH
agent or removing some keys and try again.

If the problem persists, please report a bug to the net-ssh project.

timeout during server version negotiating>
 INFO machine: Calling action: read_state on provider QEMU (HDveA27T6Xs)
 INFO interface: Machine: action ["read_state", "start", {:target=>:controlplane}]
 INFO runner: Running action: machine_action_read_state #<Vagrant::Action::Builder:0x00007fb2ed0b07a0>
 INFO warden: Calling IN action: #<Vagrant::Action::Builtin::ConfigValidate:0x00007fb2dd67e6d8>
 INFO warden: Calling IN action: #<VagrantPlugins::QEMU::Action::ReadState:0x00007fb2dd67e6b0>
 INFO warden: Calling OUT action: #<VagrantPlugins::QEMU::Action::ReadState:0x00007fb2dd67e6b0>
 INFO warden: Calling OUT action: #<Vagrant::Action::Builtin::ConfigValidate:0x00007fb2dd67e6d8>
 INFO interface: Machine: action ["read_state", "end", {:target=>:controlplane}]
DEBUG ssh: Checking key permissions: /Users/gpramita/.vagrant.d/insecure_private_key
 INFO ssh: Attempting SSH connection...
 INFO ssh: Attempting to connect to SSH...
 INFO ssh:   - Host: 127.0.0.1
 INFO ssh:   - Port: 50022
 INFO ssh:   - Username: vagrant
 INFO ssh:   - Password? false
 INFO ssh:   - Key Path: ["/Users/gpramita/.vagrant.d/insecure_private_key"]
DEBUG ssh:   - connect_opts: {:auth_methods=>["none", "hostbased", "publickey"], :config=>false, :forward_agent=>false, :send_env=>false, :keys_only=>true, :verify_host_key=>:never, :password=>nil, :port=>50022, :timeout=>15, :user_known_hosts_file=>[], :verbose=>:debug, :logger=>#<Logger:0x00007fb2ed02eae8 @level=0, @progname=nil, @default_formatter=#<Logger::Formatter:0x00007fb2ed02eac0 @datetime_format=nil>, @formatter=nil, @logdev=#<Logger::LogDevice:0x00007fb2ed02ea70 @shift_period_suffix=nil, @shift_size=nil, @shift_age=nil, @filename=nil, @dev=#<StringIO:0x00007fb2ed02eb38>, @binmode=false, @mon_data=#<Monitor:0x00007fb2ed02ea48>, @mon_data_owner_object_id=79360>>, :keys=>["/Users/gpramita/.vagrant.d/insecure_private_key"], :remote_user=>"vagrant"}
 INFO machine: Calling action: read_state on provider QEMU (HDveA27T6Xs)

```

I raised the issue https://github.com/ppggff/vagrant-qemu/issues/32 for vagrant as using the below command of qemu, the same amd64 ubuntu machine was up and running

```qemu-system-aarch64 \
        -machine virt,accel=hvf \
        -cpu host \
        -smp 8 \
        -m 8G \
        -drive if=virtio,cache=none,format=raw,file=./ubuntu.img \
        -cdrom box.img \
        -net user,hostfwd=tcp::10022-:22 -net nic -nographic \
        -bios QEMU_EFI.fd```


QEMU_EFI.fd -> curl -L https://releases.linaro.org/components/kernel/uefi-linaro/latest/release/qemu64/QEMU_EFI.fd -o QEMU_EFI.fd
box.img -> .vagrant.d/boxes/roboxes-VAGRANTSLASH-ubuntu2004/4.2.10/libvirt/box.img
