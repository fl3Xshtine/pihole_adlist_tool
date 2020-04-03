# Pihole Adlist Tool


This script tries to provide you with a bunch of information that enables you to decide which adlists you need based on your browsing behavior. It does that by matching your browsing history (FTL's querylog) with your current adlist configuration (gravity database) generating a list of domains that you have visited in the past and which would have been blocked if your current adlist configuration would have been in place back then. In a second step the scripts takes this list and attributes each domain to the adlists it is on (similar to what `pihole -q` does). The final output is a table of all your adlists with the corresponding number of covered domains (domains that you have visited and that would have been blocked if only this particular adlist would have been used).

**The script outputs**

-  the number of adlists (and how many are enabled)
- the number of unique domains in your gravity.db
- the number of blocked domains as reported by pihole ('blocking status == blocked by gravity') and how often those domains have been blocked ('hits')
- the number of covered domains and how often those would have been blocked ('hits')
- special case: domains on your (personal) blacklist which are also on an adlist and have been visited in the past, including hits (run 'pihole -q' to see on which adlist those domains appear)
- optional: top blocked domains and number of hits if your current adlist configuration would have been used
- adlist table
    id, status, total domains on adlist, covered domains, hits, unique covered domains, address
 -  the sum of unique covered domains
- optional: list of unique coverd domains with adlist_id, address

As domains usually appear on more then one adlist I introduce the concept of ***unique covered domains***. Those are domains that have been visited, would have been blocked and appear on just one adlist. This might help you to value your adlists not just by how many domains are covered but also what would happen if you disable this adlist.


**Limits**

- Disabled blocklist won't be analyzed as gravity is not including domains from deactivated adlists. If you want to check them also you have to enable them and run `pihole -g`

-  Black/Whitelisted domains (including regex) are not considered when calculating the number of covered domains (and hits)
	- Whitelisted domains reduce the number of blocked domains as reported by pihole compared to the calculated numbers
	-  Blacklisted domains increase the number of blocked domains as reported by pihole compared to the calculated numbers
	
-  This tool can not deal with domains that have been blocked due to CNAME inspection because pihole doesn't store the actual blocked domain but the CNAME and a corresponding status ("Blocked during deep CNAME inspection"). This CNAME domain will not match a domain from an adlist - if it would it would have been blocked directly.

-  Other differences between the number of domains/hits as reported by pihole and calculated numbers are due to change in adlist configuration over time


**Cave**

Depending on the number of enabled adlists and the number of visited domains in the selected time period the calculation might take some time - please be patient.


**Requirements:**

- Pihole v5.0


** Usage **

```bash	
pihole_adlist_tool [options]

options:
    -d [Num]                         Consider the last [Num] days (Default: 90). Enter 0 for all-time analysis.
    -t [Num]                         Show top blocked domains. [Num] defines the number to show.
    -s [total/domains/hits/unique]   Set sorting order to total domains, domains covered, hits covered or unique covered domains DESC. (Default sorting: id ASC)
    -u                               Show covered unique domains
    -h                               Show this help dialog

```
    


**Background**

As adlist configuration might have changed over time (add/removed adlists, enabled/disabled adlists) this script doesn't rely on pholes blocking status for the analysis but rather determine if queries from the long-term database would have been blocked with the current adlist configuration. Relying on the blocking status could lead to wrong assumptions about adlist coverage of your current adlist configuration: some domains might have been blocked in the past but wouldn't be blocked now (removed adlist) and some might be blocked now but haven't in the past (added adlist). If the adlist configuration hasn't changed over time there should be no huge difference between this approach and using pihole's blocking status.

The deeper reason for re-analyzing the queries is that this tool should help you to make predictions for the future: assuming your online behavior is rather stable over time and you analyze a long enough dataset from the past this tool will tell you which adlist might be worth keeping (because it contains a lot of covered domains) and which you could safely remove (no covered domains and/or covered domains but no unique covered domains).

