---
title: OpenBSM Parser
date:
  created: 2023-11-19
---

# OpenBSM Parser

A parser for the OpenBSM audit trail file format written in Python and utilising <a href="https://github.com/fox-it/dissect.cstruct">Fox-IT's Dissect.Cstruct</a>. The parser acts similar to OpenBSM's <a href="https://man.freebsd.org/cgi/man.cgi?query=praudit&sektion=1">`praudit(1)`</a> and is able to parse all record types found in macOS & FreeBSD. The output of the parser is an XML file which can then be easily ingested into a SIEM such as Splunk or Elastic for easier querying. The parser can currently be found under the following places: <a href="https://codeberg.org/bollos/jasper-vdhoven.github.io">Codeberg</a> & <a href="https://github.com/jasper-vdhoven/openbsm-parser">Github</a>.
