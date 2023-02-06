# setup-polaris-analysis
This github action will download the Polaris tools from given Polaris address and extract them into given location. If cache is used, then tools are not installed again, only once per given Polaris tool version. Polaris tools bin folder is set to runner PATH, so all the Polaris tools are available. Will also add some key-value pairs to environmen variables and after setup they are available for next steps in the pipeline.

## Prerequisities
This action expects that given Polaris tool version is available via /api/tools/v2/downloads/ -API from given Polaris URL.

## Available Options
| Option name | Description | Default value | Required |
|-------------|-------------|---------------|----------|
| polaris_url | URL for Polaris where the thin client can be downloaded | - | true |
| polaris_token | Polaris Access Token | - | true |
| polaris_install_folder | To which folder the tools are extracted. Default is /tmp/cache/polaris | /tmp/cache/polaris | false |
| polaris_platform | What platform of Polaris thin client is needed, example: linux64 (Default). Fow Windows win64 and for MacOS macosx | linux64 | false |
| polaris_version | What version of Polaris thin client is needed, example: 2022.9.0 (Default) | 2022.9.0 | false |
| cache | Polaris tools can be cached by setting this to true. Default is true. | true | false |

## Values which will be set into environment variables

These key-value pairs are set into environment values and are accessed with **${{env.key}}**
| Key | Value |
|----------|--------|
| POLARIS_SERVER_URL | ${{inputs.polaris_url}} |
| POLARIS_ACCESS_TOKEN | ${{inputs.polaris_token}} |
| POLARIS_HOME | ${{inputs.polaris_install_folder}} |
| POLARIS_VERSION | ${{inputs.polaris_version}} |

## Usage

**Example full pipeline:**

In this full pipeline example we first set up the Polaris tools into runner PATH with setup-polaris-analysis action and
then we are using other action called [lejouni/polaris-analysis](https://github.com/lejouni/polaris-analysis) to run the actual polaris analysis.
```yaml
name: Java CI with Maven and Polaris

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3 # This will checkout the source codes from repository

    - name: Set up JDK 1.11 # This will add Java into the runners PATH
      uses: actions/setup-java@v3.6.0
      with:
        java-version: '11'
        distribution: 'temurin'
        cache: 'maven'

    - name: Set up Polaris # This will add Polaris tools into runner PATH
      uses: synopsys-sig-community/polaris-analysis@main
      with:
        polaris_url: ${{secrets.POLARIS_SERVER_URL}} #Polaris server URL
        polaris_token: ${{secrets.POLARIS_ACCESS_TOKEN}} #Polaris Access Token
    
    - name: Analyze with Polaris
      uses: synopsys-sig-community/polaris-analysis@main
      with:
        polaris_config_file: polaris.yml
        build_command: mvn package
        polaris_sarif: true

    - name: Upload SARIF file
      uses: github/codeql-action/upload-sarif@v2
      with:
        # Path to SARIF file
        sarif_file: ${{github.workspace}}/polaris-scan-results.sarif.json
      continue-on-error: true

    - name: Archive scanning results
      uses: actions/upload-artifact@v3
      with:
        name: polaris-scan-results
        path: ${{github.workspace}}/polaris-scan-results.sarif.json
      continue-on-error: true
```

**Setup with required inputs:**

In this example is given only the required inputs.
```yaml
    - name: Set up Polaris # This will add Polaris tools into runner PATH
      uses: synopsys-sig-community/setup-polaris-analysis@main
      with:
        polaris_url: ${{secrets.POLARIS_SERVER_URL}} #Polaris server URL
        polaris_token: ${{secrets.POLARIS_ACCESS_TOKEN}} #Polaris Access Token
```

**Setup with all available inputs:**

In this example is given all available inputs.
```yaml
    - name: Set up Polaris # This will add Polaris tools into runner PATH
      uses: synopsys-sig-community/setup-polaris-analysis@main
      with:
        polaris_url: ${{secrets.POLARIS_SERVER_URL}} #Polaris server URL
        polaris_token: ${{secrets.POLARIS_ACCESS_TOKEN}} #Polaris Access Token
        polaris_install_folder: /tmp/cache/polaris
        polaris_platform: linux64
        polaris_version: 2022.9.0
        cache: true
```
