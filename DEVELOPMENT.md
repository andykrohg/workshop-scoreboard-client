# Scoreboard Client Development Guide

## Running the client locally
### Prereqs
1. Ensure you have `ansible` and python prereqs installed:
   ```bash
   pip3 install ansible kubernetes jmespath
   ```
2. Make sure you have `oc` installed.
3. Run `oc login` to authenticate to OpenShift.

### Getting started
1. Clone this repository, and `cd` into it:
   ```bash
   git clone https://github.com/andykrohg/workshop-scoreboard-client.git
   cd workshop-scoreboard-client
   ```
2. Run the `main.yml` playbook from the workshop you're using, e.g.:
   ```bash
   ansible-playbook workshops/experience-openshift-virtualization/main.yml
   ```

## Customizing the client
You can modify the client for use with other workshops by updating the playbooks, or indeed using whatever language or framework you're most comfortable with. Here's how the client works:

1. **Run in a loop** to continually scoop up updates as participants make their way along. This is currently accomplished by wrapping our playbook execution in a [bash script](workshops/loop.sh).
2. **Use RoleBindings** to ensure your client has permissions to access the data it needs to collect from your cluster. We create this as part of the [template](template.yml) that's used to deploy the client.
3. **Send a PUT Request to the scoreboard server** at the end of each loop to either register your attendee or update the status. We're using the `uri` ansible module to accomplish this in [our playbook](workshops/experience-openshift-virtualization/main.yml), and our payload looks something like this:
   ```json
   {
        "attendeeName": "Andy Krohg",
        "workshopTasks": [
            {
                "module": "0",
                "status": "success"
            },
            {
                "module": "0",
                "status": "info"
            },
            {
                "module": "0",
                "status": "pending"
            }
            ...
            {
                "module": "1",
                "status": "pending"
            }
        ]
   }
   ```
   Here's how it's constructed:
   * `attendeeName` is set to the value the attendee used for the `MY_NAME` environment variable.
   * `workshopTasks` is an ordered list of tasks, each carrying the following attributes:
     * `module`: the module identifier, which should match with the `id` in your server's `modules.json` file
     * `status`: the completion status, where `pending` is not yet started, `info` is in progress, and `success` is complete

   Once you're satisfied with your changes, proceed below to build a container for the client. You'll also need to create your own server image, instructions for which can be found [here](https://github.com/andykrohg/workshop-scoreboard-server/blob/main/DEVELOPMENT.md).

## Building
1. Build a container image with podman:
   ```bash
   podman build --platform linux/amd64 -t scoreboard-client -f Containerfile .
   ```
2. Now you're ready to push your new image to quay.io and substitute it in the deployment template [here](template.yml), or through some other means of deployment.
