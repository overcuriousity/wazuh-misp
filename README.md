# MISP SSH Brute Force Integration

A Python integration tool that aggregates SSH brute force attack data from honeypots and publishes it to a MISP (Malware Information Sharing Platform) instance. This integration collects failed authentication attempts, analyzes attack patterns, and creates standardized threat intelligence events.

## Features

- Aggregates multiple SSH brute force attempts from the same source IP
- Collects and correlates usernames used in attacks
- Extracts and merges MITRE ATT&CK techniques and tactics
- Includes geolocation data when available
- Uses a buffer period to aggregate multiple attempts before publishing
- Designed to work with Wazuh alerts

## Requirements

- Python 3.6+
- PyMISP library
- A running MISP instance
- Wazuh SIEM or compatible security system

## Installation

1. Clone this repository:
```bash
git clone https://github.com/overcuriousity/wazuh-misp
cd wazuh-misp
cp ...
```

2. Install the required dependencies:
```bash
pip install pymisp
```

3. Configure your MISP instance settings in the script:
```python
MISP_URL = 'https://your-misp-instance.com'
MISP_APIKEY = 'your-api-key'
HONEYPOT_AGENT_IDENTIFIER_STARTSWITH = 'honeypot-'
```

4. Install the script in your Wazuh integrations folder:
```bash
cp custom-misp_ssh_bruteforce-publish /var/ossec/integrations/
chmod +x /var/ossec/integrations/custom-misp_ssh_bruteforce-publish
```

## Configuration

### Wazuh Configuration

Add the following to your Wazuh `ossec.conf` file:

```xml
<integration>
  <name>custom-misp_ssh_bruteforce-publish</name>
  <hook_url>https://your-misp-instance.com</hook_url>
  <api_key>your-api-key</api_key>
  <rule_id>5503,5710,2502</rule_id>
  <alert_format>json</alert_format>
</integration>
```

## Usage

The script is designed to work automatically with Wazuh alerts. When a matching alert is triggered (authentication failures from honeypot agents), the script will:

1. Extract relevant information
2. Store the event in a local SQLite database
3. Aggregate events from the same source IP for 15 minutes
4. Publish the aggregated data to your MISP instance

### Testing

You can test the script with the following command:

```bash
python custom-misp_ssh_bruteforce-publish --test
```

This will run the script with a sample alert to verify functionality.

## How It Works

1. The script receives JSON-formatted alerts from Wazuh.
2. It filters for SSH authentication failures from agents with names starting with the honeypot prefix.
3. Events are stored in a SQLite database for a buffer period of 15 minutes.
4. Multiple attempts from the same IP address are aggregated during this period.
5. After the buffer period, the aggregated event is published to your MISP instance.
6. The event is then removed from the local database.

## Customization

You can customize the following parameters:

- Buffer period (default: 15 minutes)
- Honeypot agent identifier prefix
- MISP event categorization and threat levels

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request
