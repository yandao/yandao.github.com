---
layout: post
title: "Setup GlassFish on Ubuntu 12.04 (Precise Pangolin)"
description: ""
category: configuration 
tags: [ubuntu, java, glassfish]
---
{% include JB/setup %}

## Introduction ##

Setting up a fresh GlassFish server can be a nightmare. This post attempts to consolidate all the information from various sources, about setting up GlassFish for Java web development. For production environment, there may be additional steps involved and are not covered here.

The setup here is done on an Ubuntu 12.04 server, but it should apply for slightly older versions of Ubuntu as well.


## Install Java JDK ##

A fresh installation of an Ubuntu server does not include Java JDK by default due to some licensing issue with Oracle (I will refrain from making any comments here). The official method of installing Java JDK is by running the binaries provided from the [Oracle Java website](http://www.oracle.com/technetwork/java/javase/downloads/index.html)... Thankfully, there are always some [3rd party efforts](http://www.webupd8.org/p/ubuntu-ppas-by-webupd8.html) that make it possible to install Java JDK via apt-get. With all the reports on security vulnerabilities on Java 6, the recommended solution nowadays is to install Java 7 directly.

### The Steps ###

	$ sudo apt-get install build-essential python-software-properties unzip

	$ sudo add-apt-repository ppa:webupd8team/java
	$ sudo apt-get update
	$ sudo apt-get install oracle-java7-installer

It may be good to install OpenJDK too for testing the more "bleeding-edge" features of Java:

	$ sudo apt-get install openjdk-7-jdk

To list all the Java JDKs/JREs that are installed:

	$ sudo update-java-alternatives -l

To set Oracle Java as default:

	$ sudo update-java-alternatives -s java-7-oracle

Since we install the JDK via apt-get, we can remove it via apt-get too.

	$ sudo apt-get remove oracle-java7-installer

If you wish to remove the PPA too:

	$ sudo ppa-purge ppa:webupd8team/java


## Install GlassFish ##

GlassFish can be installed using an installer or as a standalone zip file, both of which are available from the [Downloads](http://glassfish.java.net/public/downloadsindex.html) page. For simplicity, we choose the zip file. The latest version as of this writing is 3.1.2.2. Typically, I place all the applications that can be run without root privileges in ~/opt. **From the point of security, it may be good to create a new user and group specifically for running GlassFish processes only**. The steps can be found [here](/configuration/2012/10/11/first-things-after-fresh-installation-of-a-linux-server/#add-non-root-user) and therefore will not be included in this post.

### The Steps ###

	$ mkdir -p ~/opt
	$ cd ~/opt/
	$ wget http://download.java.net/glassfish/3.1.2.2/release/glassfish-3.1.2.2.zip
	$ unzip glassfish-3.1.2.2.zip 

It is a good idea to include the bin/ path of GlassFish to the PATH environment variable, so that we can run asadmin from anywhere:

	$ vim ~/.bash_profile

		export PATH=$PATH:~/opt/glassfish3/bin


## Initial Configuration for GlassFish ###

Just a few things to do first after the fresh installation of GlassFish. This will save us from all the hair-pulling and head-banging in the future.

### Disable auto-update ###

The reason why GlassFish is always so slow when starting is because it tries to perform an automatic update first before starting the server proper. To disable this, include the following JVM option in the domain.xml file (just place it together with all the other options):

	$ vim ~/opt/glassfish3/glassfish/domains/domain1/config/domain.xml

		...

		<jvm-options>-Dcom.sun.enterprise.tools.admingui.NO_NETWORK=true</jvm-options>

		...

Now GlassFish will start up in blazing fast time, and the Admin Console will be much more responsive too.

### Change Admin password ###

Starting from version 3.1.2, any administration processes will require entering of admin username and password. I could not, for the life of me, find the default admin password for GlassFish (it is probably empty string). Anyway, this [article](https://blogs.oracle.com/dipol/entry/glassfish_3_1_2_secure) outlines the steps to change the admin password using asadmin:

	$ cd ~/opt/glassfish3/glassfish/
	$ touch /tmp/password.txt
	$ chmod 600 /tmp/password.txt 
	$ echo "AS_ADMIN_PASSWORD=" > /tmp/password.txt 
	$ echo "AS_ADMIN_NEWPASSWORD=<the_new_password>" >> /tmp/password.txt 
	$ asadmin --user admin --passwordfile /tmp/password.txt change-admin-password --domain_name domain1
	$ rm /tmp/password.txt 

Now GlassFish is set up with a new admin password.

To enable remote administration of GlassFish server:

	$ asadmin enable-secure-admin

At this point, we can start the GlassFish server already:

	$ asadmin start-domain

and watch how blazing fast it starts up without the auto-update running in the background!


## (Optional) Setup SSL connection on GlassFish ##

If you wish to set up SSL connection on GlassFish, follow this section. As with other servers, you will need to provide:

1. An SSL certificate
2. The private key that is used to sign the certificate

### Create Self-Signed SSL Certificate ###

You can pay some trusted Certificate Authority (CA) to create those two items. Otherwise, a self-signed certificate will probably suffice, depending on whether the client applications require the certificates to be issued by a trusted CA or not. We can generate a self-signed SSL certificate using [OpenSSL](http://www.openssl.org) by following these steps ([as outlined here](https://devcenter.heroku.com/articles/ssl-certificate-self)):

OpenSSL should already be bundled in on *nix systems. To check:

	$ which openssl

It should return the full path to where OpenSSL is installed. Then we can proceed to create our self-signed certificate:

	$ mkdir -p ~/.ssh
	$ cd ~/.ssh
	$ openssl genrsa -des3 -out server.orig.key 2048
	$ openssl rsa -in server.orig.key -out server.key
	$ openssl req -new -key server.key -out server.csr
	$ openssl x509 -req -days 10000 -in server.csr -signkey server.key -out server.crt

Now we have the SSL certificate (**server.crt**) and the private key (**server.key**).

### Add SSL Certificate to GlassFish Java KeyStore ###

A Java KeyStore (JKS) is a repository of security certificates, either authorisation certificates or public key certificates. We need to add our SSL certificate to GlassFish Java KeyStore so that GlassFish can handle secure connections that use the credentials contained in our SSL certificate.

Firstly, we need to convert the SSL certificate and the private key from PEM to DER format:

	$ openssl pkcs8 -topk8 -nocrypt -in server.key -inform PEM -out key.der -outform DER
	$ openssl x509 -in server.crt -inform PEM -out cert.der -outform DER

The next step is to create a new Java KeyStore from these DER-formatted certificate and key. We can use the following Java code to create it. The code listing is taken from [here](http://knowledge-oracle.blogspot.com/2009/02/import-private-key-and-certificate-in.html):


	import java.security.*;
	import java.io.IOException;
	import java.io.InputStream;
	import java.io.FileInputStream;
	import java.io.DataInputStream;
	import java.io.ByteArrayInputStream;
	import java.io.FileOutputStream;
	import java.security.spec.*;
	import java.security.cert.Certificate;
	import java.security.cert.CertificateFactory;
	import java.util.Collection;
	import java.util.Iterator;

	/**
	 * ImportKey.java
	 *
	 * This class imports a key and a certificate into a keystore
	 * ($home/keystore.ImportKey). If the keystore is
	 * already present, it is simply deleted. Both the key and the
	 * certificate file must be in DER-format. The key must be
	 * encoded with PKCS#8-format. The certificate must be
	 * encoded in X.509-format.
	 *
	 *
	 * Import key and certificate:
	 * java ImportKey YOUR.KEY.der YOUR.CERT.der
	 *
	 *
	 * Caution: the old keystore.ImportKey-file is
	 * deleted and replaced with a keystore only containing YOUR.KEY
	 * and YOUR.CERT. The keystore and the key has no password;
	 * they can be set by the keytool -keypasswd-command for setting
	 * the key password, and the keytool -storepasswd-command to set
	 * the keystore password.
	 *
	 *
	 * The key and the certificate is stored under the alias
	 * importkey; to change this, use keytool -keyclone.
	 *
	 * Created: Fri Apr 13 18:15:07 2001
	 * Updated: Fri Apr 19 11:03:00 2002
	 *
	 * @author Joachim Karrer, Jens Carlberg
	 * @version 1.1
	 **/
	public class ImportKey  {

	    /**
	     *
	     * Creates an InputStream from a file, and fills it with the complete
	     * file. Thus, available() on the returned InputStream will return the
	     * full number of bytes the file contains


	     * @param fname The filename
	     * @return The filled InputStream
	     * @exception IOException, if the Streams couldn't be created.
	     **/
	    private static InputStream fullStream ( String fname ) throws IOException {
	        FileInputStream fis = new FileInputStream(fname);
	        DataInputStream dis = new DataInputStream(fis);
	        byte[] bytes = new byte[dis.available()];
	        dis.readFully(bytes);
	        ByteArrayInputStream bais = new ByteArrayInputStream(bytes);
	        return bais;
	    }

	    /**
	     *
	     * Takes two file names for a key and the certificate for the key,
	     * and imports those into a keystore. Optionally it takes an alias
	     * for the key.
	     *
	     * The first argument is the filename for the key. The key should be
	     * in PKCS8-format.
	     *
	     * The second argument is the filename for the certificate for the key.
	     *
	     * If a third argument is given it is used as the alias. If missing,
	     * the key is imported with the alias importkey
	     *
	     * The name of the keystore file can be controlled by setting
	     * the keystore property (java -Dkeystore=mykeystore). If no name
	     * is given, the file is named keystore.ImportKey
	     * and placed in your home directory.
	     * @param args [0] Name of the key file, [1] Name of the certificate file
	     * [2] Alias for the key.
	     **/
	    public static void main ( String args[]) {

	        // change this if you want another password by default
	        String keypass = "importkey";

	        // change this if you want another alias by default
	        String defaultalias = "importkey";

	        // change this if you want another keystorefile by default
	        String keystorename = System.getProperty("keystore");

	        if (keystorename == null)
	            keystorename = System.getProperty("user.home")+
	                System.getProperty("file.separator")+
	                "keystore.ImportKey"; // especially this ;-)


	        // parsing command line input
	        String keyfile = "";
	        String certfile = "";
	        if (args.length < 2 || args.length>3) {
	            System.out.println("Usage: java comu.ImportKey keyfile certfile [alias]");
	            System.exit(0);
	        } else {
	            keyfile = args[0];
	            certfile = args[1];
	            if (args.length>2)
	                defaultalias = args[2];
	        }

	        try {
	            // initializing and clearing keystore
	            KeyStore ks = KeyStore.getInstance("JKS", "SUN");
	            ks.load( null , keypass.toCharArray());
	            System.out.println("Using keystore-file : "+keystorename);
	            ks.store(new FileOutputStream ( keystorename  ),
	                    keypass.toCharArray());
	            ks.load(new FileInputStream ( keystorename ),
	                    keypass.toCharArray());

	            // loading Key
	            InputStream fl = fullStream (keyfile);
	            byte[] key = new byte[fl.available()];
	            KeyFactory kf = KeyFactory.getInstance("RSA");
	            fl.read ( key, 0, fl.available() );
	            fl.close();
	            PKCS8EncodedKeySpec keysp = new PKCS8EncodedKeySpec ( key );
	            PrivateKey ff = kf.generatePrivate (keysp);

	            // loading CertificateChain
	            CertificateFactory cf = CertificateFactory.getInstance("X.509");
	            InputStream certstream = fullStream (certfile);

	            Collection c = cf.generateCertificates(certstream) ;
	            Certificate[] certs = new Certificate[c.toArray().length];

	            if (c.size() == 1) {
	                certstream = fullStream (certfile);
	                System.out.println("One certificate, no chain.");
	                Certificate cert = cf.generateCertificate(certstream) ;
	                certs[0] = cert;
	            } else {
	                System.out.println("Certificate chain length: "+c.size());
	                certs = (Certificate[])c.toArray();
	            }

	            // storing keystore
	            ks.setKeyEntry(defaultalias, ff,
	                           keypass.toCharArray(),
	                           certs );
	            System.out.println ("Key and certificate stored.");
	            System.out.println ("Alias:"+defaultalias+"  Password:"+keypass);
	            ks.store(new FileOutputStream ( keystorename ),
	                     keypass.toCharArray());
	        } catch (Exception ex) {
	            ex.printStackTrace();
	        }
	    }

	} // KeyStore


Save it as ImportKey.java, compile it as usual and we can start using it. A Java KeyStore identifies the certificates stored by an alias, so we need to provide some name to be used as an alias for our certificate:

	$ javac ImportKey.java
	$ java ImportKey key.der cert.der <somealias>

The new keystore will be created at ~/keystore.ImportKey. To check its contents, run:

	$ keytool -list -v -keystore ~/keystore.ImportKey

If everything looks fine, move the keystore to GlassFish config directory:

	$ mv ~/keystore.ImportKey ~/opt/glassfish3/glassfish/domains/domain1/config/

The next step is to import the contents of this new keystore to the keystore that is used by GlassFish (keystore.jks). It is always a good idea to create a copy of this keystore so that we can always revert back should something goes wrong.

	$ cd ~/opt/glassfish3/glassfish/domains/domain1/config
	$ cp keystore.jks keystore.jks.orig
	$ keytool -importkeystore -destkeystore keystore.jks -srckeystore keystore.ImportKey -alias <somealias>

Check that our SSL certificate is now included in GlassFish Java Keystore. You can identify it by its alias that you defined above:

	$ keytool -list -v -keystore keystore.jks

Since both keystores use different passwords, the next step is to make sure that all the certificates contained in the keystore have the same passwords as the keystore itself:

	$ keytool -keystore keystore.jks -keypasswd -alias <somealias>

The default password for GlassFish Java KeyStore is **changeit**. To change it:

	$ keytool -storepasswd -keystore keystore.jks

Remove the newly-created keystore since it is no longer required:

	$ rm keystore.ImportKey

Lastly, we need to set the network listener that is configured for secure connections, to use our SSL certificate, identified by its alias. By default, GlassFish defines an HTTP listener called **http-listener-2** that listens on port 8181:

	$ asadmin set configs.config.server-config.network-config.protocols.protocol.http-listener-2.ssl.cert-nickname=<somealias>

GlassFish also provides "virtual servers" that allows you to use different server configurations for different Java applications as required:

	$ asadmin create-virtual-server --hosts <domain_name_or_ip_address> --networklisteners <network_listener_names> <virtual_server_name>

Phew! That was tedious...


## Using asadmin to perform GlassFish administration tasks ##

GlassFish comes bundled with a web-based Administration Console which is accessible via https://&lt;domain_name&gt;:4848. For security reasons, you may probably want to disable it.

Another way to perform administration tasks on GlassFish is to use the **asadmin** tool, which is accessible via the command line. If you have GlassFish installed in your local machine, you can even perform admin tasks on GlassFish server installed on a remote machine, as outlined in [this article](http://bbissett.blogspot.jp/2012/01/asadmin-with-remote-glassfish.html).

As mentioned [above](#initial-configuration-for-glassfish), GlassFish will prompt you for admin password every time you want to perform some admin tasks. To override this, there are 2 options, namely, to create a password file (which is what we used when changing the admin password above), or to run the login command.

To use the password file, create a blank text file and include the following line:

	AS_ADMIN_PASSWORD=<your_admin_password>

then we can pass it as an argument in asadmin:

	$ asadmin --user admin --passwordfile <password_file> <command>

Since the password is stored in a plain-text file, it is probably not recommended to use this option. The 2nd option is to run the login command and provide the admin credentials. Once the authentication is successful, we would be able to run asadmin commands without being prompted for the password anymore (until the session expires):

	$ asadmin login

To login to a remote GlassFish server, make sure that the remote GlassFish server is configured to accept remote administration (asadmin enable-secure-admin). Then we can login by using the following command:

	$ asadmin --host <domain_name> --port 4848 --secure login

To perform admin tasks on the remote server, we must always pass the 3 arguments: host, port and secure:

	$ asadmin --host <domain_name> --port 4848 --secure <command>

Some asadmin commands that might be useful:

	$ asadmin list-applications
	$ asadmin undeploy <application_name>
	$ asadmin deploy <path_to_war_file>
	$ asadmin deploy --virtualservers <virtual_server_name> <path_to_war_file>
	$ asadmin stop-domain <domain_name>
	$ asadmin start-domain <domain_name>


At this point, you can probably start writing your Java-based web applications and deploy them on GlassFish with less hassle. Good luck!


## Reference ##

* [Installing and running Eclipse, Glassfish and Ubuntu 12.04 Precise for Web Applications](http://connectedweb.wordpress.com/2012/03/26/creating-a-simple-ejb-enterprise-javabean-on-ubuntu/)