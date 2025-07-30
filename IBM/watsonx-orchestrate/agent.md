how to create an gent?
1. create api keys:
https://api.dl.watson-orchestrate.ibm.com/instances/20250729-0043-5160-7085-e833a97e32dd
api Key: azE6dXNyXzJhNGQ4YTY1LTYyNDgtM2RlOC1iMDFjLWZiYjQ3ODRlMmZiZDpDNTNhRnc0cEZlZzNZaUZscEtON0pYTklMUFFSZ0ZXTTZpQk9MYnRZOVM0PTpaRVgw

azE6dXNyXzJhNGQ4YTY1LTYyNDgtM2RlOC1iMDFjLWZiYjQ3ODRlMmZiZDpDNTNhRnc0cEZlZzNZaUZscEtON0pYTklMUFFSZ0ZXTTZpQk9MYnRZOVM0PTpaRVgw
orchestrate env add -n watsonx_adk -u https://api.dl.watson-orchestrate.ibm.com/instances/20250729-0043-5160-7085-e833a97e32dd --api-key azE6dXNyXzJhNGQ4YTY1LTYyNDgtM2RlOC1iMDFjLWZiYjQ3ODRlMmZiZDpUa2xva2xLVFJZU2V1VjQySGpyYjQ5ZG1xMU9VMXNzaGFwb284cWZYNG1JPTo5dkF0 --type ibm_iam --activate


the use case:

customer sends an email for a complaint.

the agent get triggered by the email as scanned documents it uses llama-3-2-90b-vision-instruct model to understand the content of the email and then based on the output, the agent will call preconfigured agent
like compliant/open account/close account workflows... so basically this step will serve as auto pilot.

