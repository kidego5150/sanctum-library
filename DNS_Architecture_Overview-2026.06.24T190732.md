
## DNS and the Disaster Recovery Exercise

- Spelled "DNS" 
- Stands for "Domain Name Service" 
- Primarily uses UDP port 53 for query resolution, and TCP port 53 for backend DNS communication.
- Its primary purpose is to translate a fully-qualified domain name such as  to an IP address so the system can connect to the resolved device.  The DNS server's job is complete and it has already moved onto the next query before your system even tries to build a connection to its destination. 

### DR Test DNS servers and their functions during the DR Exercise

- CDC/CB DRT DNS servers - These are the DNS servers that host the alternate zone files for the disaster recovery exercise.  The DRT DNS server is currently configured and running with the latest DRE configuration from DRE 2025, and will be wiped/updated once the DNS Workbooks are completed and validated by the teams for this upcoming exercise.  During a Disaster Recovery Exercise, the DRT DNS servers are configured as the primary DNS resolvers on disaster test systems.  The FQDN and IP addresses are provided to the DNS team via the DR DNS Workbook which is compiled and managed by the DRE Team lead by Janice and Jeff.  These workbooks need to be completed in time for the servers to be wiped, rebuilt and prepped for the 2026 DRE activity.

### DNS Changes upcoming before the DRE 2026

- We currently have two projects that have the possibility of affecting the run-up to the DRE

	1. The PFB to GCP Forwarding Project - This project is planned to forward all sabre.com zone resolution to retrieve directly from the Sabre GCP.  This project has been on hold due to contract stipulations between AA and Sabre, and that blocker is about 80% cleared.  Monitoring and log tracking are ongoing to determine important traffic resolving through the CMO DNS servers for AA, that would require migration to another resolution route.  This may or may not be completed before the DRE.  Either way, the only Disaster Recovery process that would change is my own internal procedures during the exercise.
	2. SGCDCDRTINTDNS01 (CDC DRT DNS Server) migration to Distribution Layer for switch decommission - This project is rehoming the DRT DNS server in CDC to the Distribution switch and migrating the VLANs to consolidate hardware in order to reconcile and decommission Access switches.  - UPDATE: This is complete.
	3. Decommission of FSE and OOB DNS servers in CMO - This should not have an effect on the DRE, apart from current production DNS configurations at the server configuration layer.  
	4. There are NO expected changes to the CB DRT DNS server before the DRE.  If something is planned, I am currently unaware.

### Changes to the DR DNS Procedures for 2026

- The only changes from DRE 2025 regarding DNS are Bruce's updates to the DNS Workbook before he left.  Those have been discussed, and we are all still trying to get a handle on that institutional knowledge gap.  
- My own internal DNS processes - modification of zone files, rebuilding and reconfiguring the DRT DNS servers for the upcoming exercise, collating and scripting the configuration changes on the Production side DNS during the exercise activity, automating the DNS processes for operator or agentic workflows, etc.  These do not have a direct effect on the DRE for any non-DCNE/DNS client or team.