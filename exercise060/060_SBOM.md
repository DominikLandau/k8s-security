### Trivy
https://trivy.dev/docs/latest/getting-started/installation/

Install trivy
```
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sudo sh -s -- -b /usr/local/bin v0.70.0
```

Analyse an image
```
trivy image --format cyclonedx --output sbom.cdx.json nginx:1.29
```

Print a report
```
trivy sbom sbom.cdx.json
```

Scan for severities
```
trivy sbom --severity CRITICAL,HIGH sbom.cdx.json   
```


#### Misconfigurations
https://trivy.dev/docs/latest/guide/scanner/misconfiguration/

#### Licenses
https://trivy.dev/docs/latest/guide/scanner/license/