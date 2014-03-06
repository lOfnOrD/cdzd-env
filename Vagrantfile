# -*- mode: ruby -*-
# vi: set ft=ruby :

$script_ci = <<SCRIPT
echo Provisioning VM...
echo Installing dependencies...
apt-get update
apt-get -qy install git openjdk-7-jdk sshpass unzip
echo Installing Jenkins...
wget -O /home/vagrant/jenkins.war http://mirrors.jenkins-ci.org/war/latest/jenkins.war
mkdir -p .jenkins/plugins
wget -O /home/vagrant/.jenkins/plugins/git-client.hpi https://updates.jenkins-ci.org/download/plugins/git-client/1.6.3/git-client.hpi
wget -O /home/vagrant/.jenkins/plugins/git.hpi https://updates.jenkins-ci.org/download/plugins/git/2.0.3/git.hpi
wget -O /home/vagrant/.jenkins/plugins/scm-api.hpi https://updates.jenkins-ci.org/download/plugins/scm-api/0.2/scm-api.hpi
wget -O /home/vagrant/.jenkins/plugins/ws-cleanup.hpi https://updates.jenkins-ci.org/download/plugins/ws-cleanup/0.20/ws-cleanup.hpi
wget -O /home/vagrant/.jenkins/plugins/token-macro.hpi https://updates.jenkins-ci.org/download/plugins/token-macro/1.10/token-macro.hpi
wget -O /home/vagrant/.jenkins/plugins/config-file-provider.hpi http://mirrors.xmission.com/hudson/plugins/config-file-provider/2.7.1/config-file-provider.hpi
chown -R vagrant:vagrant /home/vagrant/.jenkins
echo Installing Artifactory...
wget -O /home/vagrant/artifactory.zip http://bit.ly/Hqv9aj
unzip /home/vagrant/artifactory.zip
chown -R vagrant:vagrant /home/vagrant/artifactory-*
echo Done.
SCRIPT

$script_lb = <<SCRIPT
echo Provisioning VM...
echo Installing dependencies...
apt-get update
apt-get -qy install haproxy
echo Configuring haproxy...
echo                                                      >> /etc/haproxy/haproxy.cfg
echo "frontend http-in"                                   >> /etc/haproxy/haproxy.cfg
echo "        bind *:9000"                                >> /etc/haproxy/haproxy.cfg
echo "        default_backend servers"                    >> /etc/haproxy/haproxy.cfg
echo                                                      >> /etc/haproxy/haproxy.cfg
echo "backend servers"                                    >> /etc/haproxy/haproxy.cfg
echo "        stats enable"                               >> /etc/haproxy/haproxy.cfg
echo "        retries 3"                                  >> /etc/haproxy/haproxy.cfg
echo "        option redispatch"                          >> /etc/haproxy/haproxy.cfg
echo "        option httpchk"                             >> /etc/haproxy/haproxy.cfg
echo "        option forwardfor"                          >> /etc/haproxy/haproxy.cfg
echo "        option http-server-close"                   >> /etc/haproxy/haproxy.cfg
echo "        server app1 10.0.2.2:9101 check inter 1000" >> /etc/haproxy/haproxy.cfg
echo "        server app2 10.0.2.2:9102 check inter 1000" >> /etc/haproxy/haproxy.cfg
echo "ENABLED=1" > /etc/default/haproxy
service haproxy restart
echo Done.
SCRIPT

$script_app = <<SCRIPT
echo Provisioning VM...
echo Installing dependencies...
apt-get update
apt-get -qy install openjdk-7-jre-headless
echo Done.
SCRIPT

$script_db = <<SCRIPT
echo Provisioning VM...
echo Installing dependencies...
apt-get update
apt-get -qy install openjdk-7-jre-headless
wget -O /home/vagrant/hsqldb.jar http://search.maven.org/remotecontent?filepath=org/hsqldb/hsqldb/2.3.1/hsqldb-2.3.1.jar
echo Done.
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "saucy-server-cloudimg-amd64"
  config.vm.box_url = "http://cloud-images.ubuntu.com/vagrant/saucy/current/saucy-server-cloudimg-amd64-vagrant-disk1.box"

  config.vm.define "ci" do |ci|
    ci.vm.hostname = "cdzero2hero-ci"
    ci.vm.provision :shell, :inline => $script_ci
    ci.vm.network "forwarded_port", guest: 22, host: 8022
    ci.vm.network "forwarded_port", guest: 8080, host: 8080
    ci.vm.network "forwarded_port", guest: 8081, host: 8081
    ci.vm.provider "virtualbox" do |vm|
      vm.customize [
                     'modifyvm', :id,
                     '--memory', '2048',
                     '--cpus', '2',
                 ]
    end
  end

  config.vm.define "lb" do |lb|
    lb.vm.hostname = "cdzero2hero-lb"
    lb.vm.provision :shell, :inline => $script_lb
    lb.vm.network "forwarded_port", guest: 9000, host: 9000
    lb.vm.provider "virtualbox" do |vm|
      vm.customize [
                     'modifyvm', :id,
                     '--memory', '512'
                 ]
    end
  end

  config.vm.define "app1" do |app1|
    app1.vm.hostname = "cdzero2hero-app1"
    app1.vm.provision :shell, :inline => $script_app
    app1.vm.network "forwarded_port", guest: 22,   host: 9111
    app1.vm.network "forwarded_port", guest: 9100, host: 9101
    app1.vm.provider "virtualbox" do |vm|
      vm.customize [
                     'modifyvm', :id,
                     '--memory', '512'
                 ]
    end
  end

  config.vm.define "app2" do |app2|
    app2.vm.hostname = "cdzero2hero-app2"
    app2.vm.provision :shell, :inline => $script_app
    app2.vm.network "forwarded_port", guest: 22,   host: 9112
    app2.vm.network "forwarded_port", guest: 9100, host: 9102
    app2.vm.provider "virtualbox" do |vm|
      vm.customize [
                     'modifyvm', :id,
                     '--memory', '512'
                 ]
    end
  end

  config.vm.define "db" do |db|
    db.vm.hostname = "cdzero2hero-db"
    db.vm.provision :shell, :inline => $script_db
    db.vm.network "forwarded_port", guest: 9200, host: 9200
    db.vm.provider "virtualbox" do |vm|
      vm.customize [
                     'modifyvm', :id,
                     '--memory', '512'
                 ]
    end
  end
end
