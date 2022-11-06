1)AwsTemplateFormatVersion --> THIS IS A STANDARD KEY --> Google it paste it --> Organize it if you need to do.

2)Under AwsTemplateFormatVersion which is into json format put the "Description": -->This is optional. It gives info what you are doing

3)Now, it is time to use your keyPair(.pem file) We will parameterized it! We use "Parameters": { "WebServerKey": { --> this is a logical ID! "Description": --> Give any description you want! "Type":"AWS::EC2::KeyPair::KeyName" }, --> don't forget the comma!! **BE CAREFUL WITH OPEN CLOSE CURL BRACES !!

4)"Resources":{} --> Now, It is time use this. when you create something from scratch , YOU MUST USE this KEY!! NOTE---> There is a main curl brace!! and inside of the main curl braces we are open new curl braces or we are using new keys

so far this is what it is { -->OPENcurlBrace A "AWSTemplateFormatVersion": "2010-09-09", "Description":"" "Parameters": { "WebServerKey": { "Description":"", "Type": "AWS::EC2::KeyPair::KeyName" } }, "Resources": { ->open curl brace for resources

    ALL COMPONENTS WILL BE CREATED HERE WITH name and into curl braces 
    as an examp-> "VPC": {},"Public1": {}, etc --> DON'T forget the comma between each 
    
    }  ->close curl brace for resources
}-->CLOSECurlBrace A

5-Create the VPC now -> template link -> https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-vpc.html
IMPORTANT: READ REQUIREMENTS CAREFULL AND USE THE KEY VALUE PAIRS WHAT ELSE YOU NEED TO!

"Tags": [
--You are giving the vpc name into "Tags":[{"Key":"NAME,"Value": "YouWriteWhatNameYouWantTO"}] { "Key": "Name", "Value": "YouWriteWhatNameYouWantTO" }

6-Now put a comma after the close curl Braces and start creating you Subnets!!!
Template Link ->https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet.html

-->"MapPublicIpOnLaunch" : true, --> IT DOES ENABLE OR DISABLE AUTO ASSIGN PUBLIC IPv4 ADDRESS!

NOTE: WE MUST referance our vpc into our subnets to DO CONNECTION! Otherwise Subnets are not going to be connected to the VPC!!!

7-Copy and Paste second public subnet into the json array. You must need to change availability zones,name,cdirBlock.

NOW, Let's go the aws UI and check it out it is woking or not!!!*** a.->CloudFormation b.->Create stack c.->choose Create template in Designer radioBox d.->click Create template in Designer e.->click Template-->IT IS ON THE BOTTOM LEFT. IT IS NEXT To COMPONENTS! d.-> Delete everthing and paste your Json array to here! e.-> Click reload icon which is right side on the top, next to ? icon f.-> If you are having an error message read it carefully and fix it. Otherwise, you will see a diagram which means that you are done if you haven't made a mistake with cidrblocks etc! h.-> Click "Check" icon(It is on the left side on the top next to "Close" icon) to validate your json array YOU WILL SEE A pop-up Message says that "Template is Valid" I.->click Cloud icon(it is next to "Check" icon) in order to create your stack!! J.Now, your stack is inside of "Amazon S3 URL" --> Click next and give a Name

*************** IT WORKED WELL. IT IS UP AND RUNNING BUT THERE IS A PROBLEM *********************** The problem is subnets are connected to only one region! What if the connected aws region is crashed/Down/Not working... our application will stop working too!!! For the Disaster recovery tool / Zero down time --> we will use "Dynamic AZ(Availabity zones)" -search for cft az dynamic on google --> Fn::GetAZs --> GezAZs is a ready Cloud Formation Function.Ready to use! Link->https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getavailabilityzones.html

--Go down on the link you will see a usage example with subnets into a Json format. copy it and paste it in to your subnets!!

8- I have created a new File ->cft_CreateVpc_DynamicAZ-> And use dynamic az's there. { "Fn::Select" : [ "0", --------> I use "0" or "1" for the ones that I want them to be in the same available zones! --------> I chose my region as nort virginia -> 0 means first index-> means "us-east-1a" --------> Why I am passing this index instead of directly writing "us-east-1a"? BECAUSE : I WANT TO USE same CFT/Stack for several different regions(Cali,Ohio etc) AND I DON"T WANT TO KEEP WRITING THEM OVER AND OVER! THIS IS MORE REUSABLE and GREAT AUTOMATION! { "Fn::GetAZs" : "" } ] }

9- Vpc and public subnets are done. Now I am creating 2 private subnets!
Private subnets --> "MapPublicIpOnLaunch" : false MUST BE FALSE!!!!! <<<<<<< HEAD

10- Create "InternetGateway" and "InternetGatewayAttachment" --> makes VPC and internetGateway attach each other!

11- Route table -> https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-routetable.html

12-Route table configuration->https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-route.html

PublicRouteConfiguration" -->"DependsOn":"InternetGatewayAttachment" it makes ->Wait until internetGateway created!

13-Now, It is time to Associations --> You must associate your subnets to your Route Table!!!! Link:https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnetroutetableassociation.html 2 public subnets' associations are done ->"PublicSubnet1RouteTableAssociation" "PublicSubnet2RouteTableAssociation"

14-Now, According to the diagram, create a NatGateway link🔗https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-natgateway.html

"AllocationId" : { "Fn::GetAtt": ["NatElasticIP","AllocationId"]}, -> will explain later

15-Now, Create an elastic IP Because your NatGateway is Public ->You must Allocate an elastic IP!! You were doing the same thing on Amazon UI too!!!!!
"NATGatewayEIP" : { "Type" : "AWS::EC2::EIP", "Properties" : { "Domain" : "vpc" } }

16-Creating Private rt and Private rt configuration(::Route) --> Important ---> Private route table should be connected to NATGateway for security reason!!! That's why we Ref-> NatGateway and  we will use "NatGatewayId" : NOT "GatewayId": in properties!

17-Now, We are creating private subnets assoasiation  as AppLayer1PrivateSubnetAssociation, AppLayer2PrivateSubnetAssociation

18- vpc-> 4 subnets-> Internet Gateway -> 2 route tables -> NAT GAteway ARE DONE!!!! CREATED!!
   I WILL DEPLOY IT BY USING AWS CLI from my Terminal!!!! NOT from AWS UI, NOT FROM Cloud Formation Stack!!!

   DEPLOYING by AWS CLI STEPS
   1)Go to IAM services on AWS UI create a user with ->Access key-Programatic access ->IT will provide you "access key ID" and "secret access key" for AWS CLI,SDK,API etc. Dowload the csv into your local!!!
   2)Go to your Bash terminal. Type aws -> make sure that you have aws cli installed.
   3)aws configure
       AWS Access Key ID [****************L76I]: Paste your Key ID here
       AWS Secret Access Key [****************/GJd]: Paste your 
       Default region name [None]: us-east-1  -> Set your region where you want to
       Default output format [None]: json     -> set the format

   4)aws sts get-caller-identity -> in order to check it out your user is Correct!
   5)https://docs.aws.amazon.com/cli/latest/reference/cloudformation/validate-template.html  --> go to get the command line IN ORDER TO VALIDATE YOUR TEMPLATE!!! 
    command:aws cloudformation validate-template --template-body file:// paste your file name here -> for this case -> cft_vpc.json
   6)aws cloudformation deploy --template-file "fileName" --stack-name "YougiveyourStackNameHere"   --> IN ORDER TO DEPLOY 
      IT GIVES AN ERROR -> [WebServerKey] must have values because we MUST give our KeyPair too!!!!

      COMMAND TO DEPLOY!!!!:aws cloudformation deploy --template-file YOURFILENAME --stack-name YOURSTACKNAME --parameter-overrides WebServerKey=YourKEYPAIR
   7)DELETE COMMAND FOR STACK -> If you want to delete your stack -> aws cloudformation delete-stack --stack-name YOURSTACKNAME 

**** CI/CD TOGETHER!!!! by Using Github! *** -> We need an EC2 for this!!! --> EC2 instance needed as an Agent!
    1)EC2 insatance exists and to be connected through Bash Terminal. 
    2)Github->Settings->Actions-> Runner -> New selfHosted runner -> If no runners are configured you need to follow the instructions in order to do the configuration!
      2.1) Choose runner image ->Linux or etc. I will choose linux for this case.
      2.2) There is a copy option next to each download commadn but I will paste it here just in case step by step
          2.2.1)mkdir actions-runner && cd actions-runner                  --> actions-runner is a generic name of the folder . You can type what you want to!!!!
          2.2.2)curl -o actions-runner-linux-x64-2.298.2.tar.gz -L https://github.com/actions/runner/releases/download/v2.298.2/actions-runner-linux-x64-2.298.2.tar.gz
          2.2.3)tar xzf ./actions-runner-linux-x64-2.298.2.tar.gz
      2.3) Configure Github Runner
         2.3.1)./config.sh --url https://github.com/NamikanilBasnuh/CI_CD_CFT --token APJU37TOOCSU7F7QZRD34F3DM4PEY 
         2.3.2) Click Enter for default settings until you see
               √ Runner successfully added
               √ Runner connection is good
               √ Settings Saved.
         2.3.3)./run.sh   --> you will see √ Connected to GitHub if everyhing is good. And you will see "Listening for Jobs" in the end!!!

    3) Create a workflow folder as ->mkdir -p .github/workflows
        -p means: create the directory and, if required, all parent directories
    4)You need a yml file inside of .github/workflows and write all runs and configurations there!!!
      4.1) While putting you keys here :
                - name: Configure AWS credentials
                  uses: aws-actions/configure-aws-credentials@v1
                  with:
                    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}             --> YOU MUST USE GITHUB SECRETS!!!!!
                    aws-secret-access-key: ${{ secrets.AWS_ACCESS_SECRET_KEY_ID }}  --> YOU MUST USE GITHUB SECRETS!!!!!
                    aws-region: us-east-1
    5)Go to Github -> go to your repository that you want to do the CI/CD -> Settings -> Secrets -> Actions -> repositery secrets -> new repository secret
                   ->   Name -> AWS_ACCESS_KEY_ID