##### Apt Mirros

1. mkdir -p /var/lib/libvirt/images/apt01/

2. wget -O /root/create-config-drive https://raw.githubusercontent.com/Mirantis/mcp-common-scripts/2018.4.0/config-drive/create_config_drive.sh
chmod +x /root/create-config-drive


wget -O /root/user_data.sh https://raw.githubusercontent.com/Mirantis/mcp-common-scripts/2018.4.0/config-drive/mirror_config.sh


export MCP_VERSION="2018.4.0"
https://raw.githubusercontent.com/Mirantis/mcp-common-scripts/${MCP_VERSION}/predefine-vm/define-vm.sh
