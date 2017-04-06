# Running Asadmin Commands on Bootstrapped Instances using the API

It is possible to execute administration commands programmatically using the [Payara Micro API](/documentation/payara-micro/appendices/micro-api.md), using the `PayaraMicroRuntime.run` method. There are two overloaded forms that can be used:

The **first**, `run(String command, String... args )`, executes the specified _asadmin_ command on all instances in the runtime's cluster. It returns a `Map<InstanceDescriptor, Future<ClusterCommandResult>>`, which can be used to inspect the results of the execution on each instance the command was executed.

The **second**, `run(Collection<InstanceDescriptor> members, String command, String... args )`, executes the specified _asadmin_ command on all instances contained in the `Collection` supplied. As with the previous method, it returns a `Map<InstanceDescriptor, Future<ClusterCommandResult>>`, which can be used to inspect the results of the command execution. 

For both methods, it's recommended to use the `getClusteredPayaras()` method exposed by the `PayaraMicroRuntime` class to retrieve a list of all the instances in the cluster and determine which command executions should be filtered.

Here's an example of how to use the first form of the method that executes the `deploy` subcommand to deploy an application and parses the results to check out if the application was succesfully deployed on all instances of the cluster:

 ```Java
import fish.payara.micro.BootstrapException;
import fish.payara.micro.ClusterCommandResult;
import fish.payara.micro.PayaraMicro;
import fish.payara.micro.PayaraMicroRuntime;
import fish.payara.micro.data.InstanceDescriptor;
import java.util.Map;
import java.util.concurrent.Future;
import java.util.logging.Logger;

public class EmbeddedPayara {

    private static final Logger LOG = Logger.getLogger(EmbeddedPayara .class.getName());

    public static void main(String[] args) throws BootstrapException {
    
        PayaraMicro instance = PayaraMicro.getInstance();
        PayaraMicroRuntime runtime = instance.bootStrap();
        
        Map<InstanceDescriptor, Future<? extends ClusterCommandResult>> results = runtime.run("deploy", "application.war");
        for(InstanceDescriptor descriptor : results.keySet()){
            ClusterCommandResult commandResult = (ClusterCommandResult)results.get(descriptor);
            if(commandResult.getExitStatus() == ClusterCommandResult.ExitStatus.SUCCESS){
                LOG.info("Application was succesfully deployed on cluster instance " + descriptor.getInstanceName());
            }
        }
    }
}
```