# Test the local macOS runner

The manual `Self-hosted Runner Smoke Test` workflow verifies that GitHub can run
a job on the Apple Silicon Mac and that the job can read the local Minikube
nodes. It does not deploy the application or publish an image.

The Solar System workflow uses GitHub-hosted Ubuntu runners for its test jobs
because their MongoDB service containers and coverage job container require
Linux runners. Its Docker build and push job runs on this self-hosted runner.

## Register and start the runner

1. Open the repository's Settings > Actions > Runners > New self-hosted runner.
2. Select macOS and ARM64. Follow GitHub's download and configuration commands
   in a dedicated directory outside the repository.
3. Register the runner as your regular macOS user, the one whose kubeconfig is
   at `/Users/lamda/.kube/config`. Add the custom label `minikube` when prompted.
   Keep the default `self-hosted`, `macOS`, and `ARM64` labels.
4. From the runner directory, run `./run.sh` and leave that terminal open for
   this first test. Confirm that GitHub shows the runner as Idle.
5. Keep the Mac awake and Minikube running. Confirm local access:

   ```bash
   kubectl --context minikube --request-timeout=15s get nodes
   ```

No kubeconfig GitHub secret or inbound network port is required. The job uses
the runner user's existing kubeconfig and certificate files on the Mac.

## Run the test

Make `.github/workflows/self-hosted-smoke-test.yml` available on the repository's
default branch, then open Actions > Self-hosted Runner Smoke Test > Run workflow.
Select the branch and run it manually. This workflow has no push trigger.

Success means both steps pass: the first reports the Mac runner identity and
the second lists the Minikube node. The node should have status `Ready`.
The smoke test checks read access; it does not test deployment permissions.

If the job stays queued, confirm the runner is online and has all four labels.
If the runner step passes but the Minikube step fails, check the runner's macOS
user, the kubeconfig at `$HOME/.kube/config`, and that Minikube is running.
The workflow adds the standard Homebrew binary directories to its PATH.

## Optional background service after the test

Stop `./run.sh` with Ctrl-C, then run these commands from the runner directory
as the same macOS user, without sudo:

```bash
./svc.sh install
./svc.sh start
./svc.sh status
```

A service does not keep the Mac awake or start Minikube automatically.

References: [GitHub runner requirements](https://docs.github.com/en/actions/reference/runners/self-hosted-runners)
and [macOS runner service setup](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/configure-the-application?platform=mac).
