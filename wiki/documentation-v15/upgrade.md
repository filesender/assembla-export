
      
      
      
      
      
      
      
      
      
      
      
      
      
      <p> This page outlines the upgrade notes for the FileSender <b>1.5 release</b><br></p><h2>Upgrading to FileSender 1.5</h2><p>These upgrade notes document upgrading from 1.0.1, 1.1 and 1.1.1 to version 1.5.  If you are currently running a 1.5 beta, you can follow these notes but please also consult the <a href="https://www.assembla.com/spaces/file_sender/wiki/Installation_notes_for_1-5_development_code">Installation notes for 1-5 development code</a> </p><p><br></p><h2>What changed?</h2><ul><li><b>the client-side UI is rewritten to HTML + JavaScript.</b>  Unless uploading with a legacy browser, Flash is no longer needed</li><li>version 1.5 uses the HTML5 FileAPI for uploads larger than 2GB, <b>Gears is no longer needed</b></li><li>non-HTML5 browsers (typically older browsers) fall back to a small (invisible) Flash component for uploading, the UI is the same as for HTML5 uploads.  Please note Flash upload is limited to max. 2GB.</li><li>FileSender 1.5 introduces a database abstraction layer to be able to use
 either PostgreSQL or MySQL based on PDO (available on most default PHP 
installs)</li><li>the upload process has been made more robust, necessitating a change in the database definitions</li><li>the FileSender 1.5 web UI is multi-language.  Distribution-provided language files are found in the /language directory.  You can create local overrides in your config directory.</li><li> all local customisation now takes place in the config directory.  Files in this directory are not changed on upgrades with the exception for config-dist.php.</li><li>the distribution no longer comes with a config.php, you need to create an initial working config.php from the config/config-dist.php template <br></li><li>several configuration directives have been added, others removed, and the semantics and/or syntax of several others has changed</li><li>chunk size is now a configuration file option.  $config["upload_chunk_size"]  = '2000000';//</li><li><b>default FileSender logging now only logs on errors</b>.  Your log directory should now be empty. <br></li></ul><h2>The upgrade process summarized</h2><p>You can use the <b>same database</b> and <b>file store you used</b> for previous 
FileSender versions, but you'll have to update the database definitions.</p><p><br></p><p>What you'll need to do to upgrade is:<br></p><ol><li>backup your database</li><li>backup your filesender installation, especially if you've made local modifications to pages or code</li><li>upgrade the database definitions</li><li>install the 1.5 code from packages or by unpacking the tarball<br></li><li>create a fresh configuration file with sane defaults from the config/config-dist.php template<br></li><li>make local modifications, pay special attention to changed config file directive semantics, and the help and about <br></li></ol><h2>Upgrade your database definitions</h2>You can use your existing database but you'll need to update some database field definitions:<ul><li>To prevent problems with files sent to mutiple recipients exceeding (when combined) 250
characters, manually alter the fileto and logto types to TEXT.</li><li>Adjust
the column sizes for 'fileauthuseruid' and (if not already done)
'fileip6address' in the 'files' database table.</li></ul><p> </p><p>The changes are: <br></p><pre><code>   #sudo -u postgres psql filesender<br><span>    </span>ALTER TABLE files ALTER fileto TYPE text;<br><span>    </span>ALTER TABLE logs ALTER logto TYPE text;<br></code><code><code><span>    </span>ALTER TABLE files ALTER fileauthuseruid TYPE character varying(500);
<span>    </span>ALTER TABLE files ALTER fileip6address TYPE character varying(45);</code><br>\q</code></pre><p>When you're done the <b>files</b> and <b>logs</b> tables should look like this:<br></p><pre><code>filesender=# \d <br>                 List of relations<br> Schema |       Name       |   Type   |   Owner    <br>--------+------------------+----------+------------<br> public | files            | table    | filesender<br> public | files_fileid_seq | sequence | filesender<br> public | log_id_seq       | sequence | filesender<br> public | logs             | table    | filesender<br><br>filesender=# \d files<br>                                           Table "public.files"<br>      Column       |            Type             |                       Modifiers                        <br>-------------------+-----------------------------+--------------------------------------------------------<br> fileto            | text                        | <br> filesubject       | character varying(250)      | <br> filevoucheruid    | character varying(60)       | <br> filemessage       | text                        | <br> filefrom          | character varying(250)      | <br> filesize          | bigint                      | <br> fileoriginalname  | character varying(500)      | <br> filestatus        | character varying(60)       | <br> fileip4address    | character varying(24)       | <br> fileip6address    | character varying(45)       | <br> filesendersname   | character varying(250)      | <br> filereceiversname | character varying(250)      | <br> filevouchertype   | character varying(60)       | <br> fileuid           | character varying(60)       | <br> fileid            | integer                     | not null default nextval('files_fileid_seq'::regclass)<br> fileexpirydate    | timestamp without time zone | <br> fileactivitydate  | timestamp without time zone | <br> fileauthuseruid   | character varying(500)      | <br> filecreateddate   | timestamp without time zone | <br> fileauthurl       | character varying(500)      | <br> fileauthuseremail | character varying(255)      | <br>Indexes:<br>    "files_pkey" PRIMARY KEY, btree (fileid)<br><br>filesender=# \d logs<br>                                       Table "public.logs"<br>     Column     |            Type             |                    Modifiers                     <br>----------------+-----------------------------+--------------------------------------------------<br> logid          | integer                     | not null default nextval('log_id_seq'::regclass)<br> logfileuid     | character varying(60)       | <br> logtype        | character varying(60)       | <br> logfrom        | character varying(250)      | <br> logto          | text                        | <br> logdate        | timestamp without time zone | <br> logtime        | time with time zone         | <br> logfilesize    | bigint                      | <br> logfilename    | character varying(250)      | <br> logsessionid   | character varying(60)       | <br> logmessage     | text                        | <br> logvoucheruid  | character varying(60)       | <br> logauthuseruid | character varying(500)      | <br>Indexes:<br>    "logs_pkey" PRIMARY KEY, btree (logid)<br>
</code></pre><h2> </h2><h2>./config directory</h2><p> As of 1.5-beta2 the ./config directory as distributed only contains a 
sample config-dist.php file. All other templates/sample files have been 
moved to ./config-templates or their contents have been merged into the 
./language/* files. This means that when unpacking a release file won't 
overwrite existing configuration files. To get a working configuration 
copy config-dist.php to config.php and make the appropriate changes in 
config.php.</p><p> </p><h2>Avoid surprises and start with a fresh configuration file<br></h2><p>New configuration directives have been added, others depcrecated and at least one changed semantics.  Avoid surprises and start with a fresh configuration file.  <b>Copy config/config-dist.php to config/config.php.  This will ensure you start with sane defaults.<br></b></p><br><h2>Configuration file changes you need to be aware of</h2><p> </p><p>Check the <a href="https://www.assembla.com/spaces/file_sender/wiki/Administrator_reference_manual">Administrator Reference Guide</a> for details about the individual directives.<br></p><h4>New: <br></h4><pre><span>    </span>$config['site_defaultlanguage'] = 'en_AU';   <br><span>    </span>$config['max_html5_upload_size'] = '107374182400'; // 100  GB<br><span>    </span>$config["upload_chunk_size"]  = '2000000';//<br><span>    </span>$config["displayerrors"] = false; // Display debug errors on screen (true/false)<br><br>For those who want extra security:<br><span>    </span>$config['cron_shred'] = false; // instead of simply unlinking, overwrite expired files so they are hard to recover<br><span>    </span>$config['cron_shred_command'] = '/usr/bin/shred -f -u -n 1 -z'; // overwrite once (-n 1) with random data, once with zeros (-z), then remove (-u)<br></pre><p></p><p></p><p></p><h4 id="64656661756c74206c616e6775616765203c62723e">Default language <br>
</h4><p>The previously unused $config['site_defaultlanguage']  setting 
is as of 1.5-beta2 used to set a default language when the browser has 
an unsupported language preference. The (new) default is 'en_AU'. <br></p><pre>$config['site_defaultlanguage']  = 'en_AU' ;</pre><h4 id="configuration_of_database_parameters">Configuration of database parameters</h4><p>Configuration
 of the database connection parameters in config.php has changed to a 
database-type independent method. The old PostgreSQL-specific pg_* 
settings are now obsolete and must be replaced with their new 
equivalents (db_*) including a specification of the database type:  </p><pre>   $config["db_type"] = "pgsql";// pgsql or mysql<br>   $config['db_host'] = 'localhost';<br>   $config['db_database'] = 'filesender';<br>   $config['db_port'] = '5432';<br>   // database username and password<br>   $config['db_username'] = 'filesender';<br>   $config['db_password'] = 'yoursecretpassword';</pre><h4>Changed semantics:</h4><pre><span>    </span>$config['datedisplayformat'] = "d-m-Y";  CHANGED SEMANTICS: now only used for emails!  <br>about, help<br></pre><p>The $config['datedisplayformat'] is at this moment still used for 
formatting the exipry date of a file/voucher in outgoing mails.<br></p>Note: The display format of dates has been changed to use the <a href="http://www.php.net/manual/en/function.date.php" target="_blank">PHP-format</a>
 (instead of the previously used Flex format). The old format used in 
1.1 was DD-MM-YYYY which when used in 1.5 will give you something like 
'WedWed-SepSep-2011201120112011' . Please keep this in mind when 
transferring a custom format from a config.php in 1.0.x or 1.5 svn 
versions dated before August 24th 2011. <br><h4>Moved to the language files: </h4><pre><span>    </span>$config["AuP_label"] -> $lang["_ACCEPTTOC"]<span><br><span>    </span></span>$config["AuP_terms"] -> $lang["_AUPTERMS]"<br><span>    </span>$config["site_splashtext"] -> $lang["_SITE_SPLASHTEXT "]<br></pre><h4 style="font-weight: normal;" id="3c623e61626f75742c2068656c7020616e642061757020746578743c2f623e"><b>About, Help and AuP text</b></h4><p>As of <a href="https://www.assembla.com/code/file_sender/subversion/changesets/858">revision:858</a>
 the semantics of aboutURL and helpURL in the config.php have changed.  
They can now point to a *page* resulting in a new page being opened in a
 new tab, or they can be empty, resulting in a popup. The new defaults 
as of 1.5 are the empty value ("")  Note that the popups are "not popups
 as per opening a popup window but modal inline popups that have nothing
 to do with popup blockers.". The popup is populated from the language 
file, using _HELP_TEXT and _ABOUT_TEXT.  You can have a localised 
override in your config directory [TBD].</p><p> </p><p>Relevant config.php directives:</p><pre>   $config['aboutURL'] = "";<br>   $config['helpURL'] = "";</pre><p>Relevant settings in language files: </p><pre>   $lang["_HELP_TEXT"]<br>   $lang["_ABOUT_TEXT"]<br>   $lang["_AUPTERMS"]</pre><h4>Deprecated:</h4><pre>$config['max_gears_upload_size'] = '107374182400'; // 100  GB<br>$config['gearsURL'] = 'http://html5test.com/';<br></pre><p><br></p><h4>No longer used: temp directory</h4>as of revision:871 the temp directory is no longer used for FileSender specific actions.  This means the config directive $config["site_temp_filestore"]; is no longer used.  If you have configured PHP to use the previously configured site_temp_filestore you can keep that directory and it will still be used for Flash uploads. The setting and use of that directory is now however only defined in the site-wide PHP settings. It is still advised to keep that directory on the same disk partition as the FileSender filestore.<br><h2>Date format configuration</h2><p> There are three places where date formats are configured.  If you want to customise the date format make sure to change <i><b>all three</b></i> to ensure a consistent date format experience for your users. </p><ul><li>config.php: $config['datedisplayformat'] = "d-m-Y"; // used for date format in emails</li><li>language file (e.g. en_AU.php):<span> </span>$lang['datedisplayformat'] = "d/m/Y"; // Format for displaying date/time, use PHP date() format string syntax </li><li>language file (e.g. en_AU.php): $lang["_DP_dateFormat"] = 'dd/mm/yy';</li></ul><p><br></p><h2>Dealing with language files</h2><p>Version 1.5 supports multiple language files.  Please check the <a href="https://www.assembla.com/spaces/file_sender/wiki/Administrator_reference_manual">Administrator Reference Guide</a> for details about <a href="https://www.assembla.com/spaces/file_sender/wiki/Administrator_reference_manual#5._dealing_with_language_files_and_localisation">dealing with language files</a> and <a href="https://www.assembla.com/spaces/file_sender/wiki/Administrator_reference_manual#4._customising_the_look_and_feel_of_your_filesender_installation">customisation</a></p><p><br></p><h3 id="-_debian_.deb">Debian .deb and moving from SimpleSAMLphp 1.6.3 to 1.8.x+<br></h3><p>The unstable and testing 
repositories have SimpleSAMLphp 1.8.x packages. As of version 1.7.0 the 
Debian SimpleSAMLphp packaging switched the default baseurl from 
'simplesaml/' to 'simplesamlphp/' to adhere to standard Debian policies.
 This change is also refelected in the FileSender packages in testing 
and unstable. This might break upgrades from previous versions that were
 installed with SimpleSAMLphp version 1.6.3. Please check the relevant 
settings in /etc/simplesamlphp/config.php, /etc/filesender/config.php 
and your webserver config (Alias statements). </p><p><br></p><br>
    
    
    
    
    
    