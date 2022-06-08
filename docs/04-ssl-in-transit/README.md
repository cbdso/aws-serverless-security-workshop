# Module 4: Use SSL in-transit for your DB connections

Although we are using VPC and traffic is private within it, some regulations or compliance requirements might require encryption in transit. This encryption secures the data when communicating with the database. 

Go to *dbUtils.js* to add a new property to your database connection. Under the method ***getDbConfig***, within the resolve object (a JSON object), add a new line to the JSON:

```
    ssl: "Amazon RDS",

```
The resolve should be like this:

<details>
<summary><strong>If you haven't gone through AWS Secrets Manager step</strong></summary><p>

```javascript
			resolve({
			    ssl: "Amazon RDS",
			    host: host,
			    user: "admin",
			    password: "Corp123!",
			    database: "unicorn_customization",
			    multipleStatements: true
			});
```
</details>

<details>
<summary><strong>If you have gone through AWS Secrets Manager step</strong></summary><p>

```javascript
            client.getSecretValue({SecretId: secretName}, function (err, data) {
                if (err) {
                    console.error(err);
                    if (err.code === 'ResourceNotFoundException')
                        reject("The requested secret " + secretName + " was not found");
                    else if (err.code === 'InvalidRequestException')
                        reject("The request was invalid due to: " + err.message);
                    else if (err.code === 'InvalidParameterException')
                        reject("The request had invalid params: " + err.message);
                    else
                        reject(err.message);
                }
                else {
                    if (data.SecretString !== "") {
                        secret = data.SecretString;
                        resolve({
                            ssl: "Amazon RDS",
                            host: JSON.parse(secret).host,
                            user: JSON.parse(secret).username,
                            password: JSON.parse(secret).password,
                            database: "unicorn_customization",
                            multipleStatements: true
                        });
                    } else {
                        reject("Cannot parse DB credentials from secrets manager.");
                    }
                }
            });
```
</details>

Finally, deploy these changes:

```bash
cd ~/environment/aws-serverless-security-workshop/src
aws cloudformation package --output-template-file packaged.yaml --template-file template.yaml --s3-bucket $BUCKET --s3-prefix securityworkshop --region $REGION &&  aws cloudformation deploy --template-file packaged.yaml --stack-name CustomizeUnicorns --region $REGION --capabilities CAPABILITY_IAM --parameter-overrides InitResourceStack=Secure-Serverless
```

Once this is done, you should be able to connect to the database using SSL.

## Ensure SSL - Optional step (CONSIDER REMOVE??? STACK DOES NOT INCLUDE encrypted_user account)

You can require SSL connections for specific users accounts\. For example, you can use one of the following statements on the user account `encrypted_user`\.


```
GRANT SELECT ON *.* TO 'encrypted_user'@'%';
SET PASSWORD FOR 'encrypted_user'@'%' = PASSWORD('Corp123!');    
```


Exit the SQL connection for admin and attempt to establish an unencrypted connection with the encrypted_user account with the following command. Replace the Aurora endpoint with the one the primary instance endpoint copied into your scratch pad from the previous step.

`mysql -h <YOUR-AURORA-PRIMARY-INSTANCE-ENDPOINT> -u encrypted_user -p`

You should be prompted with a password. Use `Corp123!`
This connection attempt should work. Type `exit` to drop the mysql connection.
	
After entering your password, it should fail with ERROR 1045 (28000): Access denied for user 'encrypted_user'@'10.0.1.156' (using password: YES). This is because an encrypted connection is expected and required for this account.

**PW Notes: Verify SSL Login Steps--may need to check if complete**
Connect to your database this time using encryption with the following command. Replace the Aurora endpoint with the one the primary instance endpoint copied into your scratch pad from Step 5.

mysql -h <YOUR-AURORA-PRIMARY-INSTANCE-ENDPOINT> -u encrypted_user --ssl-ca=/home/ec2-user/environment/aws-serverless-security-workshop/src/app/assets/rds-ca-2019-root.pem -p

You should be prompted with a password. Use Corp123! After entering your password, it should login successfully and present with a mysql> prompt.

## Next step 
You have now further secured your data by enabling encryption in transit for your database connection! 

Return to the workshop [landing page](../../README.md) to pick another module.
