---
layout: default
built_from_commit: 217f9f045824d95847bfb820dffb69ce7e7b8783
title: 'Man Page: puppet certificate'
canonical: "/puppet/latest/man/certificate.html"
---

<div class='mp'>
<h2 id="NAME">NAME</h2>
<p class="man-name">
  <code>puppet-certificate</code> - <span class="man-whatis">Provide access to the CA for certificate management.</span>
</p>

<h2 id="SYNOPSIS">SYNOPSIS</h2>

<p>puppet certificate <var>action</var> [--terminus TERMINUS]
[--extra HASH]
<var>--ca-location LOCATION</var></p>

<h2 id="DESCRIPTION">DESCRIPTION</h2>

<p>This subcommand interacts with a local or remote Puppet certificate
authority. Currently, its behavior is not a full superset of <code>puppet
cert</code>; specifically, it is unable to mimic puppet cert's "clean" option,
and its "generate" action submits a CSR rather than creating a
signed certificate.</p>

<h2 id="OPTIONS">OPTIONS</h2>

<p>Note that any setting that's valid in the configuration
file is also a valid long argument, although it may or may not be
relevant to the present action. For example, <code>server</code> and <code>run_mode</code> are valid
settings, so you can specify <code>--server &lt;servername></code>, or
<code>--run_mode &lt;runmode></code> as an argument.</p>

<p>See the configuration file documentation at
<a href="https://docs.puppetlabs.com/puppet/latest/reference/configuration.html" data-bare-link="true">https://docs.puppetlabs.com/puppet/latest/reference/configuration.html</a> for the
full list of acceptable parameters. A commented list of all
configuration options can also be generated by running puppet with
<code>--genconfig</code>.</p>

<dl>
<dt>--render-as FORMAT</dt><dd>The format in which to render output. The most common formats are <code>json</code>,
<code>s</code> (string), <code>yaml</code>, and <code>console</code>, but other options such as <code>dot</code> are
sometimes available.</dd>
<dt>--verbose</dt><dd>Whether to log verbosely.</dd>
<dt class="flush">--debug</dt><dd>Whether to log debug information.</dd>
<dt>--ca-location LOCATION</dt><dd><p>Whether to act on the local certificate authority or one provided by a
remote puppet master. Allowed values are 'local' and 'remote.'</p>

<p>This option is required.</p></dd>
<dt>--extra HASH</dt><dd>A terminus can take additional arguments to refine the operation, which
are passed as an arbitrary hash to the back-end.  Anything passed as
the extra value is just send direct to the back-end.</dd>
<dt>--terminus TERMINUS</dt><dd><p>Indirector faces expose indirected subsystems of Puppet. These
subsystems are each able to retrieve and alter a specific type of data
(with the familiar actions of <code>find</code>, <code>search</code>, <code>save</code>, and <code>destroy</code>)
from an arbitrary number of pluggable backends. In Puppet parlance,
these backends are called terminuses.</p>

<p>Almost all indirected subsystems have a <code>rest</code> terminus that interacts
with the puppet master's data. Most of them have additional terminuses
for various local data models, which are in turn used by the indirected
subsystem on the puppet master whenever it receives a remote request.</p>

<p>The terminus for an action is often determined by context, but
occasionally needs to be set explicitly. See the "Notes" section of this
face's manpage for more details.</p></dd>
</dl>


<h2 id="ACTIONS">ACTIONS</h2>

<ul>
<li><p><code>destroy</code> - Delete a certificate.:
<code>SYNOPSIS</code></p>

<p>puppet certificate destroy [--terminus TERMINUS]
[--extra HASH]
<var>--ca-location LOCATION</var>
<var>host</var></p>

<p><code>DESCRIPTION</code></p>

<p>Deletes a certificate. This action currently only works on the local CA.</p>

<p><code>RETURNS</code></p>

<p>Nothing.</p></li>
<li><p><code>find</code> - Retrieve a certificate.:
<code>SYNOPSIS</code></p>

<p>puppet certificate find [--terminus TERMINUS]
[--extra HASH]
<var>--ca-location LOCATION</var>
<var>host</var></p>

<p><code>DESCRIPTION</code></p>

<p>Retrieve a certificate.</p>

<p><code>RETURNS</code></p>

<p>An x509 SSL certificate.</p>

<p>Note that this action has a side effect of caching a copy of the
certificate in Puppet's <code>ssldir</code>.</p></li>
<li><p><code>generate</code> - Generate a new certificate signing request.:
<code>SYNOPSIS</code></p>

<p>puppet certificate generate [--terminus TERMINUS]
[--extra HASH]
<var>--ca-location LOCATION</var>
[--dns-alt-names NAMES]
<var>host</var></p>

<p><code>DESCRIPTION</code></p>

<p>Generates and submits a certificate signing request (CSR) for the
specified host. This CSR will then have to be signed by a user
with the proper authorization on the certificate authority.</p>

<p>Puppet agent usually handles CSR submission automatically. This action is
primarily useful for requesting certificates for individual users and
external applications.</p>

<p><code>OPTIONS</code>
<var>--dns-alt-names NAMES</var> -
A comma-separated list of alternate DNS names for Puppet Server. These are extra
hostnames (in addition to its <code>certname</code>) that the server is allowed to use when
serving agents. Puppet checks this setting when automatically requesting a
certificate for Puppet agent or Puppet Server, and when manually generating a
certificate with <code>puppet cert generate</code>.</p>

<p>In order to handle agent requests at a given hostname (like
"puppet.example.com"), Puppet Server needs a certificate that proves it's
allowed to use that name; if a server shows a certificate that doesn't include
its hostname, Puppet agents will refuse to trust it. If you use a single
hostname for Puppet traffic but load-balance it to multiple Puppet Servers, each
of those servers needs to include the official hostname in its list of extra
names.</p>

<p><strong>Note:</strong> The list of alternate names is locked in when the server's
certificate is signed. If you need to change the list later, you can't just
change this setting; you also need to:</p>

<ul>
<li>On the server: Stop Puppet Server.</li>
<li>On the CA server: Revoke and clean the server's old certificate. (<code>puppet cert clean &lt;NAME></code>)</li>
<li>On the server: Delete the old certificate (and any old certificate signing requests)
from the <a href="https://docs.puppetlabs.com/puppet/latest/reference/dirs_ssldir.html">ssldir</a>.</li>
<li>On the server: Run <code>puppet agent -t --ca_server &lt;CA HOSTNAME></code> to request a new certificate</li>
<li>On the CA server: Sign the certificate request, explicitly allowing alternate names
(<code>puppet cert sign --allow-dns-alt-names &lt;NAME></code>).</li>
<li>On the server: Run <code>puppet agent -t --ca_server &lt;CA HOSTNAME></code> to retrieve the cert.</li>
<li>On the server: Start Puppet Server again.</li>
</ul>


<p>To see all the alternate names your servers are using, log into your CA server
and run <code>puppet cert list -a</code>, then check the output for <code>(alt names: ...)</code>.
Most agent nodes should NOT have alternate names; the only certs that should
have them are Puppet Server nodes that you want other agents to trust.</p>

<p><code>RETURNS</code></p>

<p>Nothing.</p></li>
<li><p><code>info</code> - Print the default terminus class for this face.:
<code>SYNOPSIS</code></p>

<p>puppet certificate info [--terminus TERMINUS]
[--extra HASH]
<var>--ca-location LOCATION</var></p>

<p><code>DESCRIPTION</code></p>

<p>Prints the default terminus class for this subcommand. Note that different
run modes may have different default termini; when in doubt, specify the
run mode with the '--run_mode' option.</p></li>
<li><p><code>list</code> - List all certificate signing requests.:
<code>SYNOPSIS</code></p>

<p>puppet certificate list [--terminus TERMINUS]
[--extra HASH]
<var>--ca-location LOCATION</var></p>

<p><code>DESCRIPTION</code></p>

<p>List all certificate signing requests.</p>

<p><code>RETURNS</code></p>

<p>An array of #inspect output from CSR objects. This output is
currently messy, but does contain the names of nodes requesting
certificates. This action returns #inspect strings even when used
from the Ruby API.</p></li>
<li><p><code>sign</code> - Sign a certificate signing request for HOST.:
<code>SYNOPSIS</code></p>

<p>puppet certificate sign [--terminus TERMINUS]
[--extra HASH]
<var>--ca-location LOCATION</var>
[--[no-]allow-dns-alt-names]
<var>host</var></p>

<p><code>DESCRIPTION</code></p>

<p>Sign a certificate signing request for HOST.</p>

<p><code>OPTIONS</code>
<var>--[no-]allow-dns-alt-names</var> -
Whether or not to accept DNS alt names in the certificate request</p>

<p><code>RETURNS</code></p>

<p>A string that appears to be (but isn't) an x509 certificate.</p></li>
</ul>


<h2 id="EXAMPLES">EXAMPLES</h2>

<p><code>generate</code></p>

<p>Request a certificate for "somenode" from the site's CA:</p>

<p>$ puppet certificate generate somenode.puppetlabs.lan --ca-location remote</p>

<p><code>sign</code></p>

<p>Sign somenode.puppetlabs.lan's certificate:</p>

<p>$ puppet certificate sign somenode.puppetlabs.lan --ca-location remote</p>

<h2 id="NOTES">NOTES</h2>

<p>This subcommand is an indirector face, which exposes <code>find</code>, <code>search</code>, <code>save</code>,
and <code>destroy</code> actions for an indirected subsystem of Puppet. Valid termini for
this face include:</p>

<ul>
<li><code>ca</code></li>
<li><code>disabled_ca</code></li>
<li><code>file</code></li>
<li><code>rest</code></li>
</ul>


<h2 id="COPYRIGHT-AND-LICENSE">COPYRIGHT AND LICENSE</h2>

<p>Copyright 2011 by Puppet Inc.
Apache 2 license; see COPYING</p>

</div>
