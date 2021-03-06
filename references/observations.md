# Observations of the xDscWebService PullServer (v2)

Here are some observations that I found when testing against a running xDscWebService PullServer

## References

Here are some useful site/doc references:
* Configuring LCM:
  * https://msdn.microsoft.com/en-us/powershell/dsc/metaconfig
* Setting up a pull client using configuration names:
  * https://msdn.microsoft.com/en-us/powershell/dsc/pullclientconfignames
* Setting up a pull client using configuration ID:
  * https://msdn.microsoft.com/en-us/powershell/dsc/pullclientconfigid

## Observations

* When using Configuration Names:
  * you *must* use RegistrationKeys (since Config Names are guessable)
  * you *should* specify at least one `ConfigurationName` when configuring the LCM,
    otherwise the pull server always shows the node is update-to-date (i.e. `OK`)
  * if you use multiple `ConfigurationName`s then you *must* also specify
    `PartialConfiguration` blocks in the configs
  * PullServer saves the list of configuration names from when node registered
    via `RegisterDscAgent` so that subsequent `GetDscAction` will be able to
    enumerate them all with their checksums

* When using Configuration Name with **only 1** config name:
  * The node does *not* specify the config name in `GetDscAction` requests, and
    leaves this element field empty, it does however populate the
    `Checksum` and `ChecksumAlgorithm` with its single config checksum data

* When using Configuration Names with **multiple** config names:
  * ***TODO: Test/Observe this scenario's behavior***  

* When using Configuration ID:
  * Tug does not have a design goal to support this (e.g., does not support v1 protocol)
  * LCM will issue v1.x calls to the PullServer even though it will claim
    `ProtocolVersion` = 2.0 in the request headers (this is a logged bug)
  * There is no complement to the v2 `RegisterDscAgent` in v1.x setup
    * When issuing `Set-DscLocalConfigurationManager` to enable local LCM config
      for a v1 (ConfigurationID) setup, there is no initial call from node to server
    * When issuing `Set-DscLocalConfigurationManager` to enable local LCM config
      for a v2 (ConfigurationNames) setup, the node issues a `RegisterDscAgent`
      call to the server and provides the list of config names as well as a bunch
      of node meta data (IP Addresses (all), hostname, and node certificate)

* Authorization is more or less up to the Pull server, regardless what the node sends. For example, the native Windows pull server, in v2 protocol mode, uses RegistrationKey for initial node authorization, but does not rely on it past that point. Instead, it only accepts non-registration requests from "known" nodes. The Azure Automation pull server grabs the client certificate information and _at the Web server level_ demands client authentication after the initial registration. An on-prem pull server could opt to ignore all of that and authenticate against (for example) a table of known node MAC addresses (a la 802.1X). So authorization is a funtion of the pull server.
