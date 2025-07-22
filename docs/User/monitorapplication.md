# Monitor a running App

## Check runtime policies

After an application has been deployed, ICOS will start to manage the application at runtime, by collecting several data about the application status. ICOS provides multiple ways to see the information about the running application. This is particularly useful for debugging the status of the applications and the bheaviour of ICOS itself.


The main access point to the status of ICOS resources and Applications is the **ICOS Shell**:

- in the CLI, it is possible to use the `get deployment` command:
   ```bash
   ./icos-shell --config config.yamkl get deployment --id <unique id>
   ```

- in the GUI, it is possible to see all the deployed applications in the **Deployments** section


In addition, only if the [Telemetry Dashboards](../../Administration/ICOS%20Controller/telemetry-dashboards/) are enabled (disabled by default), it is possible to have a summary of the most imporant metrics collected by ICOS for a given application in the [**ICOS App View**](../../Administration/ICOS%20Controller/telemetry-dashboards/#icos-app-view-dashboard) dashboard.


Finally, only if the Policy Manager GUI is exposed, the status of the policies can be inspected. Please refer to this page for more information: [Policy Manager GUI](../../Developer/Components/policy-manager-gui/).
