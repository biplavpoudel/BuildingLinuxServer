$ORIGIN example.com.
$TTL	1w
example.com.	IN	SOA	ns1.example.com. hostmaster.example.com. (
			3		; Serial
			1w		; Refresh
			1d		; Retry
			28d		; Expire
			1w) 	; Negative Cache TTL
			 
; name servers - NS records
		IN	NS	ns1.example.com.

; name servers - A records
ns1.example.com.		IN	A	10.0.2.5

; 10.0.2.0/24 - A records
dhcp1.example.com.		IN	A	10.0.2.4
id1.example.com.		IN	A 	10.0.2.6