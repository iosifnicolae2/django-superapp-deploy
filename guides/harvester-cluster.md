## Harvester
1. Make sure that after installing Harvester, you configure all network interfaces to have persistent names:
```bash
# Define the output file for the udev rules
output_yaml_file="/oem/99_persistent_network_interfaces.yaml"

# Start with a clean YAML file
echo "name: Persist network interfaces" > "$output_yaml_file"
echo "stages:" >> "$output_yaml_file"
echo "  rootfs:" >> "$output_yaml_file"
echo "    - files:" >> "$output_yaml_file"
echo "        - path: /etc/udev/rules.d/70-persistent-ipoib.rules" >> "$output_yaml_file"
echo "          permissions: 384" >> "$output_yaml_file"
echo "          owner: 0" >> "$output_yaml_file"
echo "          group: 0" >> "$output_yaml_file"
echo "          content: |" >> "$output_yaml_file"
echo "            # This file was generated by a script to make network interface names persistent" >> "$output_yaml_file"
echo "            # Each line corresponds to a network interface" >> "$output_yaml_file"

# Loop over each network interface that is of type PCI
for interface in $(ls /sys/class/net)
do
    # Check if the interface is a PCI device
    if [[ -d "/sys/class/net/$interface/device" ]] && [[ -e "/sys/class/net/$interface/device/vendor" ]]; then
        # Retrieve attributes to generate a unique name
        mac_address=$(cat "/sys/class/net/$interface/address")
        bus_info=$(basename $(realpath "/sys/class/net/$interface/device"))

        # Write the rule to the YAML file, indented correctly for the YAML content block
        echo "            ACTION==\"add\", SUBSYSTEM==\"net\", DRIVERS==\"?*\", ATTR{address}==\"$mac_address\", ATTR{type}==\"1\", KERNELS==\"$bus_info\", NAME=\"$interface\"" >> "$output_yaml_file"
    fi
done

# Print completion message
echo "Udev rules have been written to $output_yaml_file"
```