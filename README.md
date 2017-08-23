# mutt_ldap_query
mutt_ldap_query - Query LDAP server for Mutt mail-reader
## Synopsys
    mutt_ldap_query.pl [options] <name_to_query> [[<other_name_to_query>] ...]
## Options

* `--config=config_file` or `-c config_file` specify an alternate resource file other than the system ones (`/etc/lbdb_ldap.rc` or `/etc/mutt_ldap_query.rc`) or default personal ones (`$HOME/.lbdb/ldap.rc` or `$HOME/.mutt_ldap_query.rc`).
* `--ssl` use ssl connection
* `--server=ldap_server` or `-ls ldap_server` hostname of your ldap server.
* `--search_base=ldap_search_base` or `-sb ldap_search_base` use `<search_base>` as the starting point for the search instead of the default.
* `--search_fields=ldap_search_fields` or `-sf ldap_search_fields` list of the fields on which the query will be performed.
* `--expected_answers=ldap_expected_answers` or `-ea ldap_expected_answers` list of the fields expected as the answer of the ldap server that will be used for composing the output of the script.
* `--format_email=result_format_email` or `-fe result_format_email` format to be used for composing the email output result. It has to be based on the expected ldap server answers and can use variable containers of the form `${variable}` where variable belongs to the `<ldap_expected_answers>` set.
* `--format_realname=result_format_realname` or `-fr result_format_realname` format to be used for composing the realname output result. It has to be based on the expected ldap server answers and can use variable containers of the form `${variable}` where variable belongs to the `<ldap_expected_answers>` set.
* `--format_comment=result_format_comment` or `-fc result_format_comment` format to be used for composing the comment output result. It has to be based on the expected ldap server answers and can use variable containers of the form `${variable}` where variable belongs to the `<ldap_expected_answers>` set.
* `--bind_dn=bind_distinguished_name` or `-bd bind_distinguished_name` the distinguished name of the user who binds to the LDAP server.  Leave it empty for an anonymous bind.
* `--bind_password=secret` or `-bp secret` the bind password for binding to the LDAP server. Leave it empty for an anonmyous bind.
* `--nickname=ldap_server_nickname` or `-n ldap_server_nickname` shortcut for avoiding to use all the previous options by using the script builtin or alternate config file table of common servers and associated options.  All the required parameters are then derived by performing a `<lbdb_server_nickname>` lookup.
* `--debug` or `-d` turn on debugging messages.
* `--help` or `-?` or `-h` or `--man` or `-m` generates this help message.
* `--ignorant` or `-i` ignorant mode: search using wildcard for *name_to_query* (requires a longer processing from LDAP server but is quite convenient :).
* `--lbdb_output` or `-l` suppress number of matches output (suited for interfacing with little brother database http://www.spinnaker.de/lbdb/).
* `--version` or `-v` show the version.

## Description

`mutt_ldap_query` performs ldap queries using either `ldapsearch` command or the perl-ldap module and it outputs the required formatted data for feeding mutt when using its "External Address Query" feature.

The output of the script consists in 3 fields separated with tabs: the email address, the name of the person and a comment.

## Interfacing with mutt

This perl script can be interfaced with mutt by defining in your `.muttrc`:

    set query_command = "mutt_ldap_query.pl '%s'"

Multiple requests are supported: the "Q" command of mutt accepts as argument a list of queries (e.g. "Gosse de\ Courville").

Alternatively mutt_ldap_query can be interfaced with the more generic little brother database query program (http://www.spinnaker.de/lbdb/) using:

    set query_command = "lbdbq '%s'"

and by specifying in your `~/.lbdb/lbdbrc` file another method of query just adding to the METHODS variable the m_ldap module e.g.:

    METHODS='m_inmail m_passwd m_ldap m_muttalias m_finger'

and the right path to access m_ldap in MODULES_PATH, e.g. if you moved `m_ldap` in `~/.lbdb/modules`:

    MODULES_PATH="/usr/local/lib $HOME/.lbdb/modules"

Just make sure to use the correct path for calling mutt_ldap_query in the m_ldap script.

## Resource file format

mutt_ldap_query is now fully customizable using an external resource file. By default mutt_ldap_query parses the system definition file located generally at `/etc/mutt_ldap_query.rc` or `/usr/local/etc/mutt_ldap_query.rc` and also the user one: `$HOME/.mutt_ldap_query.rc`.

Instead of using command line options, the user can redefine all the variables using the resource file by two manners in order to match his site configuration.  A file example is provided below:

    # The format of each entry of the ldap server database is the following:
    # LDAP_NICKNAME => ['LDAP_SERVER',
    #                   'LDAP_SEARCH_BASE',
    #                   'LDAP_SEARCH_FIELDS',
    #                   'LDAP_EXPECTED_ANSWERS',
    #                   'LDAP_RESULT_EMAIL',
    #                   'LDAP_RESULT_REALNAME',
    #                   'LDAP_RESULT_COMMENT'],

    # a practical illustrating example being:
    #  debian	=> ['db.debian.org',
    #               'ou=users,dc=debian,dc=org',
    #               'uid cn sn ircnick',
    #               'uid cn sn ircnick',
    #               '${uid}@debian.org',
    #               '${cn} ${sn}',
    #               '${ircnick}'],
    # the output of the query will be then:
    # ${uid}@debian.org\t${cn} ${sn}\t${ircnick} (i.e.: email name comment)

    # warning this database will erase default script builtin
    %ldap_server_db = (
      'four11'		=> ['ldap.four11.com', 'c=US', 'givenname sn cn mail', 'givenname cn sn mail o', '${mail}', '${givenname} ${sn}', '${o}' ],
      'infospace'	=> ['ldap.infospace.com', 'c=US', 'givenname sn cn mail', 'givenname cn sn mail o', '${mail}', '${givenname} ${sn}', '${o}' ],
      'whowhere'	=> ['ldap.whowhere.com', 'c=US', 'givenname sn cn mail', 'givenname cn sn mail o', '${mail}', '${givenname} ${sn}', '${o}' ],
      'bigfoot'		=> ['ldap.bigfoot.com', 'c=US', 'givenname sn cn mail' , 'givenname cn sn mail o' , '${mail}' , '${givenname} ${sn}', '${o}' ],
      'switchboard'	=> ['ldap.switchboard.com', 'c=US', 'givenname sn cn mail' , 'givenname cn sn mail o', '${mail}', '${givenname} ${sn}', '${o}' ],
      'infospacebiz'	=> ['ldapbiz.infospace.com', 'c=US', 'givenname sn cn mail', 'givenname cn sn mail o', '${mail}', '${givenname} ${sn}', '${o}' ],
    );

    # hostname of your ldap server
    $ldap_server = 'ldap.four11.com';
    # ldap base search
    $search_base = 'c=US';
    # list of the fields that will be used for the query
    $ldap_search_fields = 'givenname sn cn mail';
    # list of the fields that will be used for composing the answer
    $ldap_expected_answers = 'givenname sn cn mail o';
    # format of the email result based on the expected answers of the ldap query
    $ldap_result_email = '${mail}';
    # format of the realname result based on the expected answers of the ldap query
    $ldap_result_realname = '${givenname} ${sn}';
    # format of the comment result based on the expected answers of the ldap query
    $ldap_result_comment = '(${o})';

## Examples of query

    mutt_ldap_query.pl --ldap_server='ldap.mot.com' --search_base='ou=employees, o=Motorola,c=US' --ldap_search_fields='commonName gn sn cn uid' --ldap_expected_answers='gn sn preferredRfc822Recipient ou c telephonenumber' --ldap_result_email='${preferredRfc822Recipient}' --ldap_result_realname='${gn} ${sn}' --ldap_result_comment='(${telephonenumber}) ${ou} ${c}' Gosse de\ Courville

performs a query using the ldap server ldap.mot.com using the following searching base 'ou=employees, o=Motorola,c=US' and performing a search on the fields 'commonName gn sn cn uid' for 'Gosse' and then "de Courville" looking for the following answers 'gn sn preferredRfc822Recipient ou c telephonenumber'. Based on this answers, mutt_ldap_query will return a list of entries identified of the form:

    <${preferredRfc822Recipient}>\t${gn} ${sn}\t(${telephonenumber}) ${ou} ${c}

where `${}` variables should be considered as containers that are replaced by the results of the query. The previous query can be greatly simplified by using the ldap server mini database feature of the resource file introducing for example a nickname.

    mutt_ldap_query.pl --ldap_server_nickname='motorola' Gosse de\ Courville

When not sure of the full name (i.e. it should contain Courville) the ignorant mode is useful since the query will be performed using wildcards, i.e. *Courville* in the following case:

    mutt_ldap_query.pl --ignorant Courville

## Where to get it

The latest version can be retrieved at ftp://ftp.mutt.org/pub/mutt/contrib or http://www.courville.org/ or https://github.com/courville./mutt_ldap_query

Note that now the script is integrated in the latest version of the little brother database available at http://www.spinnaker.de/lbdb/.  It is thus easier to use through this standard package than to hand customize it to fit your system/distribution needs.

## References

* perl-ldap module http://perl-ldap.sourceforge.net/
* mutt is the ultimate email client http://www.mutt.org/
* historical Brandon Blong's "External Address Query" feature patch for mutt http://www.fiction.net/blong/programs/mutt/#query
* little brother database is an interface query program for mutt that allow multiple searches for email addresses based on external query scripts just like this one 8-) http://www.spinnaker.de/lbdb/

## Past contributions

* 20061009: patch from Roland Kammerer to perform query to ldapssl servers
* 20060607: patch from Felice Carraro to handle multiple email answers
* 20060503: patch from Sebastien Gross to handle authenticated ldap requests

## Authors

Marc de Courville <marc@courville.org> and the various other contributors... that kindly sent their patches.

Please report any bugs, or post any suggestions, to <marc@courville.org>.

## Copyright

Copyright (c) 1998-2003 Marc de Courville <marc@courville.org>. All rights reserved. This program is free software; you can redistribute it and/or modify it under the GNU General Public License (GPL). See http://www.opensource.org/gpl-license.html and http://www.opensource.org/.
