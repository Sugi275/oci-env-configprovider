# envprovider

要英語化

oci-go-sdkを実行するために必要な全てのクレデンシャルを、環境変数として扱うためのライブラリです。
環境変数化しているもの↓

- RSA秘密鍵
- RSAパスフレーズ
- Tenancy OCID
- User OCID
- KeyFingerprint
- Region
- CompaertmentID

# Install

```
go get -u github.com/Sugi275/oci-env-configprovider/envprovider
```

# How to use

## 環境変数設定 (Example)

bash用

```
export OCI_PrivateRSAKeyEncyped=`base64 /home/ubuntu/.oci/oci_api_key.pem`
export OCI_PrivateRSAKey_passphrase="secret"
export OCI_TenancyOCID="ocid1.tenancy.oc1..secret"
export OCI_UserOCID="ocid1.user.oc1..secret"
export OCI_KeyFingerprint="00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00"
export OCI_Region="us-phoenix-1"
export OCI_COMPARTMENT_ID="ocid1.tenancy.oc1..secret"
```

fish用

```
# base64 encoded 
set IFS
set -x OCI_PrivateRSAKeyEncoded (base64 /home/ubuntu/.oci/oci_api_key.pem)
set IFS \n" "\t

set -x OCI_PrivateRSAKey_passphrase ''
set -x OCI_TenancyOCID 'ocid1.tenancy.oc1..secret'
set -x OCI_UserOCID 'ocid1.user.oc1..secret'
set -x OCI_KeyFingerprint '00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00'
set -x OCI_Region 'us-phoenix-1'
set -x OCI_COMPARTMENT_ID 'ocid1.tenancy.oc1..secret'
```

## Go SourceCode (Example)

`envprovider.GetEnvConfigProvider()` を使用することで、環境変数から ConfigProvider を生成しています。
https://github.com/Sugi275/oci-create-dnsrecords/blob/master/ocidnstest.go

```Go
package main

import (
	"context"
	"fmt"

	"github.com/Sugi275/oci-env-configprovider/envprovider"
	"github.com/oracle/oci-go-sdk/dns"
)

func main() {
	zn := "test.enc"
	dn := "_acme-challenge.test.enc"

	client, err := dns.NewDnsClientWithConfigurationProvider(envprovider.GetEnvConfigProvider())
	if err != nil {
		panic(err)
	}

	compartmentid, err := envprovider.GetCompartmentID()
	if err != nil {
		panic(err)
	}

	// DNSのレコードを作成するパラメータを生成
	txttype := "TXT"
	falseFlg := false
	rdata := "testdayo"
	ttl := 30

	recordDetails := dns.RecordDetails{
		Domain:      &dn,
		Rdata:       &rdata,
		Rtype:       &txttype,
		Ttl:         &ttl,
		IsProtected: &falseFlg,
	}

	var recordDetailsList []dns.RecordDetails
	recordDetailsList = append(recordDetailsList, recordDetails)

	updateDomainRecordsDetails := dns.UpdateDomainRecordsDetails{
		Items: recordDetailsList,
	}

	request := dns.UpdateDomainRecordsRequest{
		ZoneNameOrId:               &zn,
		Domain:                     &dn,
		UpdateDomainRecordsDetails: updateDomainRecordsDetails,
		CompartmentId:              &compartmentid,
	}

	ctx := context.Background()
	response, err := client.UpdateDomainRecords(ctx, request)
	if err != nil {
		panic(err)
	}
	fmt.Println(response)
}
```
