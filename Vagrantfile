$vm_name = "Centos7-demo"
$vm_nic = "Intel(R) Ethernet Connection (10) I219-V"
$script = <<-SCRIPT
chmod +x /home/vagrant/rootpwd.sh
SCRIPT

Vagrant.configure("2") do |config|
  #配置系統資源
  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 2
  end
  # 指定系統OS
  config.vm.box = "generic/centos7"
  # 指定OS版本
  config.vm.box_version = "4.0.0"
  # 設定hostname
  config.vm.hostname = $vm_name
  # 設定VirtualBox網路，使用Bridge橋接網路
  config.vm.network "public_network",
   bridge: $vm_nic,
   auto_config: false
   # 使用root系統權限執行下面指令
  config.vm.provision "shell", inline: <<-SHELL
    #安裝必要Library和軟體
    sudo yum -y update
    sudo yum -y install epel-release 
    sudo yum -y install docker htop git vim wget net-tools docker-compose
    systemctl enable docker
    systemctl start docker
    #設定時區服務ntp
    yum -y install ntp
    timedatectl set-timezone Asia/Taipei
    rm /etc/localtime
    ln -s /usr/share/zoneinfo/Asia/Taipei /etc/localtime
    systemctl enable ntpd
    systemctl start ntpd
    SHELL
  # 複製本機當前目錄下的檔案到vm
  config.vm.provision "file",
    source: ".",
    destination: "~/"
  # 取得vm當前的ip並寫入.env檔，然後運行gitlab docker-compose
  config.vm.provision "shell", inline: <<-SHELL
    #取得系統的ip
    IP=`ifconfig |grep 192 |awk -F ' ' '{print $2}'`
    #將ip寫入.env，以便提供環境變數給docker-compose
    echo "IP=$IP" >> /home/vagrant/.env
    #確認docker-compse是否有取得.env環境變數IP
    docker-compose config
    #啟動gitlab容器
    docker-compose up -d
    echo "Gitlab Web URL = http://${IP}:5678"
    echo "You cat exec vagrant ssh and run sudo /home/vagrant/rootpwd.sh find gitlab root password"
    SHELL
  #執行script變數的內容
  config.vm.provision "shell", inline: $script
end
