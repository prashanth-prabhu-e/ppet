Generate the VPN gateway CA:
sudo ipsec pki --gen --outform pem > vpnGWCAKey.pem

sudo ipsec pki --self --in vpnGWCAKey.pem --dn "C=US, O=Elastica, CN=Elastica VPN GW CA" --ca --outform pem > vpnGWCACert.pem
 

Generate the identity CA:
sudo ipsec pki --gen --outform pem > idCAKey.pem

sudo ipsec pki --self --in idCAKey.pem --dn "C=US, O=Elastica, CN=Elastica CA" --ca --outform pem > idCACert.pem

Generate VPN gateway server certificate:
sudo ipsec pki --gen --outform pem > vpnGWKey.pem

sudo ipsec pki --pub --in vpnGWKey.pem | ipsec pki --issue --cacert vpnGWCACert.pem --cakey vpnGWCAKey.pem  --dn "C=US, O=Elastica, CN=vpn.elastica.net" --san="vpn.elastica.net"  --flag serverAuth --flag ikeIntermediate --outform pem > vpnGWCert.pem
(Make sure to change the CN and SAN values. It should match the DNS name of the VPN server. If you plan to host the VPN gateway on your local machine, then you will need to setup DNS server with a record for the VPN server name)

Generate identity certificate (in production, we don't have generate identity CA or issue any identity certificates:
sudo ipsec pki --gen --outform pem > johnKey.pem

ipsec pki --pub --in johnKey.pem | ipsec pki --issue --cacert idCaCert.pem --cakey idCaKey.pem --dn "C=US, O=Elastica, CN=John.doe@elastica.net" --outform pem > johnCert.pem
(the CN name should match the login user name for a tenant in elastica portal)

Generate a pkcs12 for client key and certificate which will be bundled with MDM profile pushed to the iOS device
openssl pkcs12 -export -inkey johnKey.pem -in JohnCert.pem -name "john" -certfile idCaCert.pem -caname "Elastica CA" -out johnCert.p12