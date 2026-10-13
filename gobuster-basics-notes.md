# Gobuster Basics: Web Enumeration Notes

## Introduction

I studied Gobuster in the TryHackMe Gobuster Basics room to understand how it can be used during web reconnaissance and enumeration. The room focused on three modes: directory and file enumeration, DNS subdomain enumeration, and virtual host enumeration. These modes all use a wordlist, but they test different parts of a web application or its hosting configuration.

Gobuster automates repeated requests or lookups that would be slow to perform manually. It takes candidate names from a wordlist and checks whether those names correspond to directories, files, subdomains, or virtual hosts. The results help identify additional parts of a target that may need further investigation. Finding a path or hostname does not by itself prove that it is vulnerable, and the results need to be interpreted in context.

## How Gobuster Works

Gobuster is a command-line enumeration tool. Its general command structure is the tool name, a mode, and then the options required by that mode. The mode determines what Gobuster is trying to discover. Directory mode is commonly written as `dir`, DNS enumeration uses `dns`, and virtual host enumeration uses `vhost`.

I learned to check the general help output before using a tool and to consult mode-specific help when I need to understand available options. The general help command is:

```bash
gobuster --help
```

To see options for a particular mode, I can request that mode's help. For example:

```bash
gobuster dir --help
gobuster dns --help
gobuster vhost --help
```

The most important options depend on the mode. In directory mode, `-u` supplies the target URL and `-w` supplies the wordlist path. DNS mode uses `-d` to specify the domain being tested. Virtual host mode also uses a target URL, but it tests candidate hostnames by changing the HTTP `Host` header. The exact options available can vary by Gobuster version, so the help output from the installed version is the best reference.

A wordlist is a file containing candidate names that the tool will test. The quality and size of the wordlist affect what Gobuster can discover and how long the enumeration takes. A small wordlist may finish quickly but miss paths or hostnames. A very large wordlist can generate substantial traffic and take longer to complete. Enumeration should be limited to systems I own or have explicit permission to test.

The `-t` option controls the number of concurrent threads. More threads can increase the rate of requests, but they can also create more traffic and put extra load on the target. The `--delay` option can introduce a delay between requests. The `-o` option saves results to an output file, while `--debug` can help investigate unexpected behavior. These options are useful for controlling and troubleshooting a scan rather than changing what the selected mode is designed to find.

## Directory and File Enumeration

Directory mode tests candidate paths beneath a web application's URL. Websites can organize content into paths such as `/admin/`, `/images/`, or `/blog/`. A path may lead to a directory, an application route, a file, or a server response that does not expose the expected content. The path structure can look similar to a filesystem, although a URL path does not necessarily correspond directly to a physical directory on the server.

Gobuster takes each entry in the wordlist and tests it as part of the target URL. This automates the process of checking many possible paths. A basic command follows this pattern:

```bash
gobuster dir -u http://TARGET -w /path/to/wordlist.txt
```

Here, `dir` selects directory mode, `-u` sets the URL, and `-w` points to the wordlist. The target and wordlist paths are placeholders in this example, not values from my own scan.

Directory mode can also test likely file names. The `-x` option adds specified file extensions to candidate names. For example, if I include the `js` extension, Gobuster can test names such as `admin.js` or `config.js`, depending on the words in the list. This is useful when I suspect that files of a particular type may be exposed.

```bash
gobuster dir -u http://TARGET/secret/ -w /path/to/wordlist.txt -x js
```

This example shows the general form of checking for JavaScript files under a path already discovered. The path and wordlist are illustrative. In the room, I followed this workflow by enumerating the target website first, then using the discovered `secret` path as the base for a second search that looked for a JavaScript file.

The `-r` option configures Gobuster to follow redirects. Redirects matter because a requested path may send the client to another URL instead of returning the final content directly. Whether following redirects is appropriate depends on the purpose of the scan and how the application behaves.

The room also introduced `--no-tls-validation`, the long-form option used to disable TLS certificate validation. This may be needed in a controlled lab when a target uses a certificate that the client does not trust. Disabling certificate validation removes an important certificate check, so it should not be treated as a normal default when accessing real services.

### Understanding HTTP Status Codes and Results

Gobuster reports responses for candidate paths, including HTTP status codes and other response details. A `200 OK` response generally means that the server successfully returned a response for the request, but it does not automatically mean that the content is important or sensitive. A `301` or `302` response indicates a redirect. Other status codes can indicate access restrictions, missing resources, or server errors.

I learned not to select a result based on one status code alone. A web server may redirect many paths, return a custom page for missing paths, or respond in a way that makes unrelated paths look similar. The length and content of responses, the behavior after following redirects, and whether the path can actually be accessed can all help distinguish meaningful findings from noise.

In the TryHackMe exercise, directory enumeration identified a path named `secret`. I then used that path as the base URL and searched for files with a `.js` extension. The scan showed a file named `flag.js`. The lesson was the sequence: enumerate paths on the website, inspect a promising path, and then enumerate likely files inside it.

I used `curl` to request the discovered file and display the response body in the terminal. The general form is:

```bash
curl http://TARGET/secret/flag.js
```

The `curl` command sends an HTTP request to the supplied URL and prints the response body by default. In the lab, this was used to view the contents of the discovered JavaScript file. The target URL shown above is a placeholder; it is not a claim about a live website.

## DNS Subdomain Enumeration

DNS subdomain enumeration searches for hostnames that exist beneath a domain. A subdomain may look like `shop.example.com` or `forum.example.com`. Unlike a directory such as `example.com/shop/`, a subdomain is a separate hostname and is handled through name resolution. It may point to the same IP address as the main website or to a different address.

Gobuster's `dns` mode combines candidate names from a wordlist with the domain being tested. A basic command follows this pattern:

```bash
gobuster dns -d example.com -w /path/to/wordlist.txt
```

The `dns` mode selects subdomain enumeration. The `-d` option specifies the domain, and `-w` supplies the candidate names. This differs from directory mode: directory mode tests paths after a URL, while DNS mode tests hostnames formed by placing each candidate before the domain.

DNS enumeration depends on the name-resolution environment. In the TryHackMe room, the environment instructions required changing the attack machine's DNS configuration so that the lab's domain names could be resolved using the target environment. The transcript describes editing the DNSMasq configuration and restarting the service so the new configuration would take effect. The exact configuration line and target IP are not reliably preserved in the notes, so I have not reproduced them as a command. The important point is that DNS resolution had to be configured for the lab before the intended hostnames could be resolved correctly.

DNS names are case-insensitive, so capitalization does not create a separate subdomain. During the room, the output included names whose only apparent difference was capitalization. Those should be understood as the same DNS name rather than counted as separate subdomains.

In the exercise, Gobuster DNS mode identified several names associated with the lab domain, including the main `www` host and other hosts such as `forum`, `store`, and `primary`. The exercise counted distinct names rather than treating differently capitalized versions as separate results. These were findings from the controlled TryHackMe environment, not claims about an external production domain.

## Virtual Host Enumeration

A virtual host, or vhost, allows a web server to serve more than one website or site configuration. A web server can receive requests at one IP address and use the HTTP `Host` header to determine which website the client is requesting. This means different hostnames can lead to different content even when the requests reach the same server.

Virtual host enumeration is related to subdomain enumeration, but it tests a different mechanism. DNS enumeration checks whether candidate names can be resolved through DNS. Vhost enumeration sends HTTP requests and changes the hostname in the `Host` header to see whether the server responds differently for a candidate host. A hostname discovered by vhost enumeration is not necessarily a publicly configured DNS subdomain.

Gobuster's `vhost` mode can be used with a target URL and a wordlist. In the room's configuration, the domain and append-domain options were used so that each wordlist entry was combined with the lab domain. The general pattern is:

```bash
gobuster vhost -u http://TARGET -w /path/to/wordlist.txt -d example.com --append-domain
```

The URL specifies the web server to contact. The wordlist supplies candidate hostname prefixes. The `-d` option supplies the domain string, and `--append-domain` appends that domain to each candidate when constructing the hostnames to test. The URL and domain options serve different purposes: the URL identifies the server receiving the requests, while the domain value helps form the hostnames tested in the `Host` header.

The room also used `--exclude-length` to suppress responses whose body length fell within a specified range. This can reduce repetitive results when the server returns a similar default response for hostnames that do not map to a meaningful virtual host. Response length filtering is only a heuristic. A legitimate virtual host could return a response of the excluded length, and unrelated responses can have different lengths, so the results still need to be interpreted carefully.

In the lab, the vhost scan was configured to exclude a response-length range and returned four relevant hostnames with HTTP status code `200`. The result was specific to the lab's server configuration and filtering choices. A `200` response was a useful clue, but it was not sufficient by itself to prove that every result represented a distinct, useful website.

## Preparing the Lab Environment

The DNS setup section helped me understand that enumeration results depend on the environment in which the tool runs. A domain name is not automatically resolvable just because I know its text. The machine needs a DNS resolver that can provide an answer for that name. In a training lab, the intended DNS records may be available only through the lab's configured resolver or target environment.

The notes describe opening the DNSMasq configuration with elevated privileges, adding a name-resolution setting for the lab, saving the file, and restarting the DNSMasq service. `sudo` runs a command with elevated privileges, and `nano` is a terminal text editor. Restarting a service after changing its configuration allows the service to load the updated settings. Because the exact setting is not fully captured in the transcript, I would check the room's current instructions before repeating this setup rather than guessing the configuration syntax.

## What I Learned

The main distinction I took from the room is that directory enumeration, DNS subdomain enumeration, and virtual host enumeration investigate different layers. Directory mode tests URL paths and possible files. DNS mode tests candidate hostnames through name resolution. Vhost mode tests how a web server responds when the HTTP `Host` header names different sites.

I also learned to use the help output before running a command, select a wordlist appropriate to the task, and interpret results rather than treating every response as a discovery. HTTP status codes are useful evidence, but they do not replace checking the response and understanding the server's behavior. Response-length filtering can reduce noise, but it can also hide relevant results if the filter is chosen poorly.

The practical directory-enumeration sequence in the room was to scan the website for paths, identify the `secret` directory, search inside it for JavaScript files, and request the discovered `flag.js` file with `curl`. The DNS and vhost sections demonstrated how to search for additional hostnames and how the HTTP `Host` header affects which site a server returns. These were guided activities in a TryHackMe lab, and I am documenting them as learning exercises rather than as a professional penetration test.

## Conclusion

Gobuster automates repetitive enumeration by testing candidate names from a wordlist. Its usefulness depends on selecting the right mode, supplying the correct target and inputs, configuring name resolution when the environment requires it, and interpreting responses with care. I can now distinguish the purpose of its `dir`, `dns`, and `vhost` modes and understand the basic workflow used in the room. Before using the tool outside a lab, I need to confirm authorization, check the installed version's help output, and choose request rates and wordlists that are appropriate for the target.
