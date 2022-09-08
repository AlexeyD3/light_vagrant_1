# -*- mode: ruby -*-
# vi: set ft=ruby :

current_dir = File.join(File.dirname(__FILE__)) # Defining the path to the current directory

require 'yaml' # Connect and read the yaml file with the settings

if File.file?("#{current_dir}/config/vconfig.yaml")
  vconfig = YAML.load_file("#{current_dir}/config/vconfig.yaml")
else
  raise "vconfig.yaml not found"
end

Vagrant.configure("#{vconfig["api_version"]}") do |config|

  config.vm.box = vconfig["box"]

  config.vm.network "forwarded_port", guest: vconfig["vm_forwarded_port"]["guest_port"], host: vconfig["vm_forwarded_port"]["host_port"]

  config.vm.synced_folder "data", "/vagrant_data"

  config.vm.provision "shell", inline: <<-SHELL
  
    sudo -i # We use root user to create runners correctly
    sudo apt-get update
    sudo apt install curl
    curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
    sudo apt-get install gitlab-runner

    sudo apt-get update
    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
    
    sudo groupadd docker
    sudo usermod -aG docker gitlab-runner
  
    sudo gitlab-runner register -n \
      --name #{vconfig["runner_name_1"]} \
      --url #{vconfig["gitlab_runner_url"]} \
      --registration-token #{vconfig["runner_token"]} \
      --tag-list #{vconfig["runner_tag_list_1"]} \
      --executor #{vconfig["runner_executor_1"]} \
      --docker-image "docker:stable" \
      --docker-privileged 
	  
    sudo gitlab-runner register -n \
      --name #{vconfig["runner_name_2"]} \
      --url #{vconfig["gitlab_runner_url"]} \
      --registration-token #{vconfig["runner_token"]} \
      --tag-list #{vconfig["runner_tag_list_2"]} \
      --executor #{vconfig["runner_executor_2"]} 
  SHELL
end

