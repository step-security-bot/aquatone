# AQUATONE

Aquatone is a tool for visual inspection of websites across a large amount of hosts and is convenient for quickly gaining an overview of HTTP-based attack surface.

## Installation

Pre-Requisite: NPM + GO + Ruby on Rails environment +  [Google Chrome](https://www.google.com/chrome/) or [Chromium](https://www.chromium.org/getting-involved/download-chromium) browser -- **Note:** Google Chrome is currently giving unreliable results when running in *headless* mode, so it is recommended to install Chromium for the best results.

    apt install npm nodejs chromium -y
# 
or  

    apt install npm nodejs chromium-browser chromium-browser-l10n -y

1. Install Ruby on Rails

       https://gorails.com/setup/
       
In case that this command fail

    rbenv install 2.7.1
    
Execute this one to fix the error

    cd /root/.rbenv/plugins/ruby-build && git pull && cd - && rbenv install 2.7.1
       
2. Install Nokogiri       

       https://nokogiri.org/tutorials/installing_nokogiri.html
       
3. Install Aquatone Gem

       gem install aquatone

4. Update Aquatone from Git

       cd /root && git clone https://github.com/michenriksen/aquatone.git && cd /root/aquatone 

5. Upgrade npm & nodejs

       npm -v && npm install npm@latest nodejs@latest -g && npm install -g npm-check-updates && ncu && ncu -u && ncu -g

6. Install nvm

       wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh && export NVM_DIR="$HOME/.nvm" && export NVM_DIR="$HOME/.nvm" && nvm install v10.20.1 && nvm use system && nvm ls && nvm --version && npm install



#
#
#
#
    
    
## Collectors API keys

(Optional but extremely recommended)

Some of the passive collectors will require your API keys or similar credentials in order to work. Setting these values can be done with the --set-key option:

Shodan

    aquatone-discover --set-key shodan xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

Virustotal

    aquatone-discover --set-key virustotal xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    
Censys key requirements are: censys_id and censys_secret, so it would be:

    aquatone-discover --set-key censys_id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    aquatone-discover --set-key censys_secret xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

PassiveTotal key requirements are: passivetotal_key and passivetotal_secret, so:

    aquatone-discover --set-key passivetotal_key youremail@example.com
    aquatone-discover --set-key passivetotal_secret xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    
Riddle key requirements are:    
    
    riddler_username: %EMAIL% 
    riddler_password: %ACCOUNT PASSWORD% 

All keys will be saved in ~/aquatone/.keys.yml.
    

### Usage

## Discovery
The first stage of an AQUATONE assessment is the discovery stage where subdomains are discovered on the target domain using open sources, services and the more common dictionary brute force approach:

    aquatone-discover --domain example.com

aquatone-discover will find the target's nameservers and shuffle DNS lookups between them. Should a lookup fail on the target domain's nameservers, aquatone-discover will fall back to using Google's public DNS servers to maximize discovery. The fallback DNS servers can be changed with the --fallback-nameservers option:
  
    aquatone-discover --domain example.com --fallback-nameservers 87.98.175.85,5.9.49.12

## Tuning
aquatone-discover will use 5 threads as default for concurrently performing DNS lookups. This provides reasonable performance but can be tuned to be more or less aggressive with the --threads option:
    
    aquatone-discover --domain example.com --threads 25

Hammering a DNS server with failing lookups can potentially be picked up by intrusion detection systems, so if that is a concern for you, you can make aquatone-discover a bit more stealthy with the --sleep and --jitter options. --sleep accepts a number of seconds to sleep between each DNS lookup while --jitter accepts a percentage of the --sleep value to randomly add or subtract to or from the sleep interval in order to break the sleep pattern and make it less predictable.

    aquatone-discover --domain target.com --sleep 5 --jitter 30 --wordlist /usr/share/wordlists/dnsmap.txt

Please note that setting the --sleep option will force the thread count to one. The --jitter option will only be considered if the --sleep option has also been set.

## Results
When aquatone-discover is finished, it will create a hosts.txt file in the ~/aquatone/<domain> folder, so for a scan of example.com it would be located at ~/aquatone/example.com/hosts.txt. The format will be a comma-separated list of hostnames and their IP, for example:
example.com,93.184.216.34
www.example.com,93.184.216.34
secret.example.com,93.184.216.36
cdn.example.com,192.0.2.42
...
In addition to the hosts.txt file, it will also generate a hosts.json which includes the same information but in JSON format. This format might be preferable if you want to use the information in custom scripts and tools. hosts.json will also be used by the aquatone-scan and aquatone-gather tools.

For more options see.

    aquatone-discover --help 

## Scanning
The scanning stage is where AQUATONE will enumerate the discovered hosts for open TCP ports that are commonly used for web services:

    aquatone-scan --domain example.com

The --domain option will look for hosts.json in the domain's AQUATONE assessment directory, so in the example above it would look for ~/aquatone/example.com/hosts.json. This file should be present if aquatone-discover --domain example.com has been run previously.

## Ports
By default, aquatone-scan will scan the following TCP ports: 80, 443, 8000, 8080 and 8443. These are very common ports for web services and will provide a reasonable coverage. Should you want to specifiy your own list of ports, you can use the --ports option:

    aquatone-scan --domain example.com --ports 80,443,3000,8080

Instead of a comma-separated list of ports, you can also specify one of the built-in list aliases:
small: 80, 443
medium: 80, 443, 8000, 8080, 8443 (same as default)
large: 80, 81, 443, 591, 2082, 2095, 2096, 3000, 8000, 8001, 8008, 8080, 8083, 8443, 8834, 8888, 55672

 
huge: 80, 81, 300, 443, 591, 593, 832, 981, 1010, 1311, 2082, 2095, 2096, 2480, 3000, 3128, 3333, 4243, 4567, 4711, 4712, 4993, 5000, 5104, 5108, 5280, 5281, 5800, 6543, 7000, 7396, 7474, 8000, 8001, 8008, 8014, 8042, 8069, 8080, 8081, 8083, 8088, 8090, 8091, 8118, 8123, 8172, 8222, 8243, 8280, 8281, 8333, 8337, 8443, 8500, 8834, 8880, 8888, 8983, 9000, 9043, 9060, 9080, 9090, 9091, 9200, 9443, 9800, 9981, 11371, 12443, 16080, 18091, 18092, 20720, 55672
Example:

    aquatone-scan -d yourtarget.com -p large -t 37 -s 5 -j 30

## Tuning
Like aquatone-discover, you can make the scanning more or less aggressive with the --threads option which accepts a number of threads for concurrent port scans. The default number of threads is 5.
$ aquatone-scan --domain example.com --threads 25
As aquatone-scan is performing port scanning, it can obviously be picked up by intrusion detection systems. While it will attempt to lessen the risk of detection by randomising hosts and ports, you can tune the stealthiness more with the --sleep and --jitter options which work just like the similarly named options for aquatone-discover. Keep in mind that setting the --sleep option will force the number of threads to one.

Results
When aquatone-scan is finished, it will create a urls.txt file in the ~/aquatone/<domain> directory, so for a scan of example.com it would be located at ~/aquatone/example.com/urls.txt. The format will be a list of URLs, for example:
http://example.com/
https://example.com/
http://www.example.com/
https://www.example.com/
http://secret.example.com:8001/
https://secret.example.com:8443/
http://cdn.example.com/
https://cdn.example.com/
...
This file can be loaded into other tools such as EyeWitness.
aquatone-scan will also generate a open_ports.txt file, which is a comma-separated list of hosts and their open ports, for example:
93.184.216.34,80,443
93.184.216.34,80
93.184.216.36,80,443,8443
192.0.2.42,80,8080
...

for more options see>
    
    aquatone-scan --help 

## Gathering
The final stage is the gathering part where the results of the discovery and scanning stages are used to query the discovered web services in order to retrieve and save HTTP response headers and HTML bodies, as well as taking screenshots of how the web pages look like in a web browser to make analysis easier. The screenshotting is done with the Nightmare.js Node.js library. This library will be installed automatically if it's not present in the system.
    
    aquatone-gather --domain example.com

aquatone-gather will look for hosts.json and open_ports.txt in the given domain's AQUATONE assessment directory and request and screenshot every IP address for each domain name for maximum coverage.

## Tuning
Like aquatone-discover and aquatone-scan, you can make the gathering more or less aggressive with the --threads option which accepts a number of threads for concurrent requests. The default number of threads is 5.
    
    aquatone-gather --domain example.com -t 37 -s 5 -j 37

As aquatone-gather is interacting with web services, it can be picked up by intrusion detection systems. While it will attempt to lessen the risk of detection by randomising hosts and ports, you can tune the stealthiness more with the --sleep and --jitter options which work just like the similarly named options for aquatone-discover. Keep in mind that setting the --sleep option will force the number of threads to one.

Results
When aquatone-gather is finished, it will have created several directories in the domain's AQUATONE assessment directory:
headers/: Contains text files with HTTP response headers from each web page
html/: Contains text files with HTML response bodies from each web page
screenshots/: Contains PNG images of how each web page looks like in a browser
report/ Contains report files in HTML displaying the gathered information for easy analysis

## Takeover
Now to find vulnerable Subdomain type

    aquatone-takeover --domain yourdomain.com -t 37 -s 5 -j 37

If any subdomain is vulnerable it will display cname and etc

#
#
#
#


## Usage

### Command-line options:

```
  -chrome-path string
    	Full path to the Chrome/Chromium executable to use. By default, aquatone will search for Chrome or Chromium
  -debug
    	Print debugging information
  -http-timeout int
    	Timeout in miliseconds for HTTP requests (default 3000)
  -nmap
    	Parse input as Nmap/Masscan XML
  -out string
    	Directory to write files to (default ".")
  -ports string
    	Ports to scan on hosts. Supported list aliases: small, medium, large, xlarge (default "80,443,8000,8080,8443")
  -proxy string
    	Proxy to use for HTTP requests
  -resolution string
    	screenshot resolution (default "1440,900")
  -save-body
    	Save response bodies to files (default true)
  -scan-timeout int
    	Timeout in miliseconds for port scans (default 100)
  -screenshot-timeout int
    	Timeout in miliseconds for screenshots (default 30000)
  -session string
    	Load Aquatone session file and generate HTML report
  -silent
    	Suppress all output except for errors
  -template-path string
    	Path to HTML template to use for report
  -threads int
    	Number of concurrent threads (default number of logical CPUs)
  -version
    	Print current Aquatone version
```

### Giving Aquatone data

Aquatone is designed to be as easy to use as possible and to integrate with your existing toolset with no or minimal glue. Aquatone is started by piping output of a command into the tool. It doesn't really care how the piped data looks as URLs, domains, and IP addresses will be extracted with regular expression pattern matching. This means that you can pretty much give it output of any tool you use for host discovery.

IPs, hostnames and domain names in the data will undergo scanning for ports that are typically used for web services and transformed to URLs with correct scheme.  If the data contains URLs, they are assumed to be alive and do not undergo port scanning.

**Example:**

    cat /root/aquatone/domain.com/hosts.txt | /root/aquatone/./aquatone

### Output

When Aquatone is done processing the target hosts, it has created a bunch of files and folders in the current directory:

 - **aquatone_report.html**: An HTML report to open in a browser that displays all the collected screenshots and response headers clustered by similarity.
 - **aquatone_urls.txt**: A file containing all responsive URLs. Useful for feeding into other tools.
 - **aquatone_session.json**: A file containing statistics and page data. Useful for automation.
 - **headers/**: A folder with files containing raw response headers from processed targets
 - **html/**: A folder with files containing the raw response bodies from processed targets. If you are processing a large amount of hosts, and don't need this for further analysis, you can disable this with the `-save-body=false` flag to save some disk space.
 - **screenshots/**: A folder with PNG screenshots of the processed targets

The output can easily be zipped up and shared with others or archived.

#### Changing the output destination

If you don't want Aquatone to create files in the current working directory, you can specify a different location with the `-out` flag:

    cat /root/aquatone/domain.com/hosts.txt | aquatone -out /root/aquatone/./aquatone/example.com




### Specifying ports to scan

Be default, Aquatone will scan target hosts with a small list of commonly used HTTP ports: 80, 443, 8000, 8080 and 8443. You can change this to your own list of ports with the `-ports` flag:

    cat /root/aquatone/domain.com/hosts.txt | /root/aquatone/./aquatone -ports 80,443,3000,3001

Aquatone also supports aliases of built-in port lists to make it easier for you:

 - **small**: 80, 443
 - **medium**: 80, 443, 8000, 8080, 8443 (same as default)
 - **large**: 80, 81, 443, 591, 2082, 2087, 2095, 2096, 3000, 8000, 8001, 8008, 8080, 8083, 8443, 8834, 8888
 - **xlarge**: 80, 81, 300, 443, 591, 593, 832, 981, 1010, 1311, 2082, 2087, 2095, 2096, 2480, 3000, 3128, 3333, 4243, 4567, 4711, 4712, 4993, 5000, 5104, 5108, 5800, 6543, 7000, 7396, 7474, 8000, 8001, 8008, 8014, 8042, 8069, 8080, 8081, 8088, 8090, 8091, 8118, 8123, 8172, 8222, 8243, 8280, 8281, 8333, 8443, 8500, 8834, 8880, 8888, 8983, 9000, 9043, 9060, 9080, 9090, 9091, 9200, 9443, 9800, 9981, 12443, 16080, 18091, 18092, 20720, 28017

**Example:**

    cat /root/aquatone/domain.com/hosts.txt | /root/aquatone/./aquatone -ports large


### Usage examples

Aquatone is designed to play nicely with all kinds of tools. Here's some examples:

#### Amass 

[Installation](https://github.com/4k4xs4pH1r3/Amass/blob/master/doc/install.md) 

by OWASP is currently my preferred tool for enumerating DNS. It uses a bunch of OSINT sources as well as active brute-forcing and clever permutations to quickly identify hundreds, if not thousands, of subdomains on a  domain:

### Amass DNS enumeration

```bash
$ amass enum -o /root/aquatone/domain.com/hosts.txt -d yahoo.com
alerts.yahoo.com
ads.yahoo.com
am.yahoo.com
- - - SNIP - - -
prd-vipui-01.infra.corp.gq1.yahoo.com
cp103.mail.ir2.yahoo.com
prd-vipui-01.infra.corp.bf1.yahoo.com
cat /root/aquatone/domain.com/hosts.txt | /root/aquatone/./aquatone
```

There are plenty of other DNS enumeration tools out there and Aquatone should work just as well with any other tool:

- [Sublist3r](https://github.com/4k4xs4pH1r3/Sublist3r)
- [Subfinder](https://github.com/4k4xs4pH1r3/subfinder)
- [Gobuster](https://github.com/4k4xs4pH1r3/gobuster)
- [Knock](https://github.com/guelfoweb/knock)
- [Fierce](https://www.aldeid.com/wiki/Fierce)


#### Nmap or Masscan

Aquatone can make a report on hosts scanned with the [Nmap](https://nmap.org/) or [Masscan](https://github.com/robertdavidgraham/masscan) portscanner. Simply feed Aquatone the XML output and give it the `-nmap` flag to tell it to parse the input as Nmap/Masscan XML:

    cat /root/aquatone/domain.com/scan.xml | /root/aquatone/./aquatone -nmap

#
#
#
#

### Compiling the source code

If you for some reason don't trust the pre-compiled binaries, you can also compile the code yourself. **You are on your own if you want to do this. I do not support compiling problems. Good luck with it!**

### Credits

https://github.com/michenriksen/aquatone

- Thanks to [EdOverflow](https://twitter.com/EdOverflow) for the [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz/) project which Aquatone's domain takeover capability is based on.
- Thanks to [Elbert Alias](https://github.com/AliasIO) for the [Wappalyzer](https://github.com/AliasIO/Wappalyzer) project's technology fingerprints which Aquatone's technology fingerprinting capability is based on.



