trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- task: BrowserStackConfig@0
  name: 'BrowserStackConfiguration'
  displayName: 'BrowserStack Configuration'
  inputs:
    BrowserStackServiceEndPoint: 'BrowserStack'

- task: DotNetCoreCLI@2
  name: 'RestoreProject'
  displayName: 'Restore Project'
  inputs:
    command: 'restore'
    projects: '**/*.sln'
    feedsToUse: 'select'

- task: DotNetCoreCLI@2
  name: 'BuildProject'
  displayName: 'Build Project'
  inputs:
    command: 'build'

- task: DotNetCoreCLI@2
  name: 'TestProjectBuild'
  displayName: 'Run Tests'
  inputs:
    command: 'test'
    arguments: '--filter FullyQualifiedName\!~ReportGeneration'
  continueOnError: true
  env:
    CAPABILITIES_FILENAME: capabilities.yml
    BROWSERSTACK_ANDROID_APP_ID: SimpleApp_Android
    BROWSERSTACK_IOS_APP_ID: SimpleApp_iOS
    BROWSERSTACK_APP_ID: <Test_App>
    BROWSERSTACK_BUILD_NAME: browserstack-examples-appium-nunit-$(Build.BuildId)
- task: DotNetCoreCLI@2
  name: 'TestProjectReport'
  displayName: 'Run Report'
  inputs:
    command: 'test'
    arguments: '--filter ReportGeneration'
  continueOnError: true
  env:
    BROWSERSTACK_BUILD_NAME: browserstack-examples-appium-nunit-$(Build.BuildId)

- task: BrowserStackResults@1
  inputs:
    browserstackProduct: 'app-automate'
  continueOnError: true
- task: PublishPipelineArtifact@1
  displayName: 'Publish Pipeline Artifact'
  inputs:
    targetPath: ./Reports/output.html
    artifact: 'Build Report'
