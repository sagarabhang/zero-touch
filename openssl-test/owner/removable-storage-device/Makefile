
generate_offline_data:
	@#Everything needed:
	@#+--ro (information-type)        anydata
	@#+--ro signature?                binary
	@#+--ro ownership-voucher?        binary
	@#+--ro owner-certificate?        binary
	@#+--ro voucher-revocation?       binary
	@#+--ro certificate-revocation?   binary


	@echo "\n[6.1] constructing information-type..."
	@$(eval B64 = $(shell $(OPENSSL) x509 -in ../../3rd-party-ca/tls-server-certificate-pki/trust-anchor-ca/ca_cert.pem -outform der | $(OPENSSL) enc -base64 -A))
	@sed 's#BOOSTRAP_SERVER_TRUST_ANCHOR#$(B64)#' .redirect-information-template.xml > information-type.xml

	@echo "\n[6.2] generating signature..."
	$(OPENSSL) smime -sign -signer ../owner-certificate/owner_certificate.pem -binary -in information-type.xml -inkey ../owner-certificate/private_key.pem -nocerts -noattr -nodetach -out signature.smime
	$(OPENSSL) smime -pk7out -in signature.smime | $(OPENSSL) pkcs7 -out signature.pk7 -outform DER
	@#$(OPENSSL) asn1parse -inform DER -in signature.pk7 -i
	@rm -f signature.smime

	@echo "\n[6.3] collecting ownership-voucher..."
	cp ../../device/ownership-voucher/ownership-voucher.pk7 .

	@echo "\n[6.4] collecting owner-certificate..."
	cp ../owner-certificate/owner_certificate.pk7 .

	@echo "\n[6.5] assembling voucher-revocation bundle..."
	@# already the voucher is a PKCS#7, so what we can do is add all the CRLs from the
	@# cert chain CAs into it as 'helper objects'
	cat ../../vendor/ownership-voucher-pki/intermediate2-ca/crl.pem ../../vendor/ownership-voucher-pki/intermediate1-ca/crl.pem ../../vendor/ownership-voucher-pki/trust-anchor-ca/crl.pem > tmp_crls.pem
	../../tools/pack-ovr-bundle -c tmp_crls.pem -v ../../vendor/ownership-voucher-pki/intermediate3-ca/ownership-voucher-revocation.pk7 -b ownership-voucher-revocation.pk7
	@rm tmp_crls.pem

	@echo "\n[6.6] assembling owner-certificate-revocation bundle..."
	cat ../../3rd-party-ca/owner-certificate-pki/intermediate-ca/crl.pem ../../3rd-party-ca/owner-certificate-pki/trust-anchor-ca/crl.pem > tmp_crls.pem
	../../tools/pack-crls -i tmp_crls.pem -o owner-certificate-revocation.pk7
	@rm tmp_crls.pem
	#$(OPENSSL) asn1parse -inform DER -in owner-certificate-revocation.pk7 -i


clean:
	@rm -f private_key.pem cert_request.pem owner_cert.pem
	@rm -f owner_cert.pem owner_cert.pk7
	@rm -f information-type.xml
	@rm -f signature.pk7 signature.smime
	@rm -f ownership-voucher.pk7
	@rm -f owner_certificate.pk7
	@rm -f voucher-revocation.pk7
	@rm -f owner-certificate-revocation.pk7
	@rm -f ownership-voucher-revocation.pk7


