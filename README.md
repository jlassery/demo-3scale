# Demo using Fuse Online and 3Scale 

Demo showing Fuse Online and 3Scale. This guide was based on [Rodrigo Ramalho demo](https://gist.github.com/hodrigohamalho/a52dfb24383c89b4c6b1022123e195f2)and Gustavo luszczynski

We will create and expose an API that get data from a posgresql database using Fuse Online. After that, we'll expose it through 3Scale, you will set it up on the API Management platform to have control and visibility about this API.

![](imgs/imagemh1.png)


## Links

* 3Scale Dashboard Samples: https://rramalho-admin.3scale.net
* Link fuse online

## Pre-req

* Account on 3scale: https://www.3scale.net/
* Anyone environment RHPDS with OCP **OR** the easy form: this environment have the operators of Fuse online instantiated and ready to use yet >>> RHPDS >      "Workshops > DIL Streaming - Event-driven Workshop" 

### Workshops > DIL Streaming - Event-driven Workshop”

* When instantiated will receive all credentials of environment 
* The projects are create automatically 
* This environment have the operator of Fuse in each project called "fuse-userX", just click route of "Syndesys...hproxy" and Fuse Online will open.

### Creating database on Openshift

First, login to Openshift using the credentials sent to you via e-mail, in this DEMO was used login **'admin'**
Now, let's create a new database in project `fuse-userX`: (If you prefer, can create all things via Openshift WEB)
```bash
# Create a new postgresql database using a Openshift template
oc new-app --template=postgresql-persistent --param=POSTGRESQL_PASSWORD=redhat --param=POSTGRESQL_USER=redhat --param=POSTGRESQL_DATABASE=sampledb -n fuse-demo
```
When the pod is ready, run:

```bash
# Get postgresql pod name
POD_POSTGRESQL=$(oc get po | grep postgresql | awk '{print $1}')

# Create database
oc exec -it $POD_POSTGRESQL -- bash -c 'psql -U redhat -d sampledb -c "CREATE TABLE users(id serial PRIMARY KEY,name VARCHAR (50),phone VARCHAR (50),age integer);"'

# Populate the database
oc exec -it $POD_POSTGRESQL -- bash -c "psql -U redhat -d sampledb -c \"INSERT INTO users(name, phone, age) VALUES  ('Jade', '(21) 12345678', 24);\""
oc exec -it $POD_POSTGRESQL -- bash -c "psql -U redhat -d sampledb -c \"INSERT INTO users(name, phone, age) VALUES  ('Francisco', '(11) 95474-8099', 40);\""
oc exec -it $POD_POSTGRESQL -- bash -c "psql -U redhat -d sampledb -c \"INSERT INTO users(name, phone, age) VALUES  ('Pedro', '(11) 23454367', 29);\""
oc exec -it $POD_POSTGRESQL -- bash -c "psql -U redhat -d sampledb -c \"INSERT INTO users(name, phone, age) VALUES  ('Rafael', '(21) 95474-8099', 55);\""
oc exec -it $POD_POSTGRESQL -- bash -c "psql -U redhat -d sampledb -c \"INSERT INTO users(name, phone, age) VALUES  ('Rodrigo', '(11) 95474-8099', 36);\""

# Make sure your data is saved
oc exec -it $POD_POSTGRESQL -- bash -c "psql -U redhat -d sampledb -c \"select * from users;\""
```

> If for some reason you need to reinstall the database, just run:

```bash
oc delete all -l app=postgresql-persistent -n fuse-demo
oc delete pvc postgresql -n fuse-demo
oc delete secret postgresql -n fuse-demo
```

## 1.Creating the Users API using the Low code Integration Solution Fuse Online
If you provisioned the **easy** environment have the operator of Fuse online in each project called "fuse-userX", just click route of "Syndesys...hproxy" and Fuse Online will open.

![](imgs/im1.png)
####  1.1. Creating the connection with the Users Database
Put the credentials {userX} passwd:openshift and accept the permission grant.

![](imgs/im2.png)
![](imgs/im3.png)

1. Now that you are on the iPaaS solution Red Hat Fuse, click on `Connections`

![](imgs/im4.png)

2. Click on `Create Connection`

![](imgs/im5.png)


3. Filter for `database` and select the Database connection

![](imgs/im6.png)

4. Fill the database configuration with the following values:
- Connection URL: jdbc:postgresql://postgresql.fuse-userX:5432/sampledb
- Username: redhat
- Password: redhat

![](imgs/im7.png)
Now, click on `Validate` to make sure everything is working as expected. If it is all good, click on `Next`.

5. Fill **Users Database** for the `Connection Name`, then click on `Create`

![](imgs/im8.png)

Now you should see connection **Users Database** listed in the connections page.
![](imgs/im9.png)

#### 1.2. Design and Create the Users API 
Now the we have the Users Database already configured as a valid connection, we will create the connection to interact with this database and export it as a REST API.

1. On the side menu `Integrations, select `Create Integration`

![](imgs/im10.png)

2. Select API Provider from the connections listed.

![](imgs/im11.png)

3. Choose like below


![](imgs/im12.png)






































Open Fuse Online

![](imgs/01.png)

Click on `Connections`

![](imgs/02.png)

Click on `Create Connection`

![](imgs/03.png)

Then, select `Database`

![](imgs/04.png)

Fill the database configuration with the following values:

```properties
url: jdbc:postgresql://postgresql.fuse-demo:5432/sampledb
user: redhat
password: redhat
```

![](imgs/05.png)

Now, click on `Validate` to make sure everything is working as expected. If it is all good, click on `Next`.

![](imgs/09.png)

The Connection Name is: `Users Database`. Then, click on `Create`

![](imgs/06.png)

Now you should see connection `Users Database` listed in the connections page.

![](imgs/10.png)

We are good to go for our API creation demo.

## Demo

### Create an API from Scratch

Back to our `Home` page, click on `Create Integration`

![](imgs/11.png)

Then select `API Provider` from the connections listed.

![](imgs/12.png)

Choose `Create from scratch`

![](imgs/13.png)

Click on `Add a data type`

![](imgs/14.png)

Give it a name like: `User`

![](imgs/15.png)

Paste the following json example and choose `REST Resource`. Then, click `Save`.

```json
{
    "id": 0,
    "name": "Rodrigo Ramalho",
    "phone": "11 95474-8099",
    "age": 30
}
```

Click `Save` again.

![](imgs/16.png)

Now, click on `Next`

![](imgs/17.png)

And give a name for our integration: `Users API`. Click on `Save and continue`

![](imgs/18.png)

#### Creating an API for `Get All Users` (GET)

Create now a flow for the GET Method that list all users:

![](imgs/19.png)

Add a step in our flow clicking on `+`:

![](imgs/20.png)

Now choose our `Users Database` connection created previously.

![](imgs/21.png)

Click on `Invoke SQL to obtain, store, update or delete data`:

![](imgs/22.png)

Fill the `SQL Statement` with: `select * from users` and then click `Next`

![](imgs/23.png)

Add a log step in our flow. Click again on the `+`:

![](imgs/24.png)

Then choose `Log`

![](imgs/25.png)

In the `Custom Text`, write `Loading users from database` and click `Done`.

![](imgs/26.png)

Now, let's add a data mapping to our flow. In the last step, click in the yellow icon and then go to `Add a data mapping step`.

![](imgs/27.png)

Expand both panel clicking on the arrows:

![](imgs/28.png)

Now, drag and drop the source fields matching with the target fields and then click on `Done`.

![](imgs/29.png)

Click now on `Save`.

![](imgs/30.png)

#### Creating API for `Create a users` (POST)

From the combobox `Operations`, choose `Create a users`:

![](imgs/31.png)

Repeat the same steps you did when `Creating an API for Get All Users (GET)`

When adding the Users Database, you need to click on `Invoke SQL to obtain, store, update or delete data` and add `INSERT INTO USERS(NAME,PHONE,AGE) VALUES(:#NAME,:#PHONE,:#AGE);` in the field `SQL statement`.

![](imgs/32.png)

Also, during the data mapping you won't need to associate the `id` field because it will be already generate by the postgres database.

![](imgs/33.png)

In the end, you should have something like:

![](imgs/34.png)

Now, click on `Save` and then on `Publish`

![](imgs/35.png)

Now, we need to wait Openshift build our container. When done, you should see `Published version 1` on the top of the page.

If you go to the `Home` page, we have 1 integration running.

![](imgs/37.png)

Our last step is to expose our integration on Openshift using `Route`s.

```bash
oc create route edge i-users-api --service=i-users-api -n fuse
```

### Testing your integration

You can check if your integration is working properly running:

```bash
curl https://$(oc get route -n fuse | grep i-users-api | awk '{print $2"/users"}')
```

Or you can try with [httpie](https://httpie.org/):

```bash
http https://$(oc get route -n fuse | grep i-users-api | awk '{print $2"/users"}')
```

### Exposing your API using 3Scale

#### Importing API from Openshift

First, let's import our API from Openshift. To do that, just click on `NEW API`.

![](imgs/38.png)

Select `Import from Openshift`. Then choose `fuse` for the `Namespace` combobox and `i-users-api` for the `Name` field. Click on `Create Service`.

![](imgs/39.png)

Now you should see your new api on the 3scale dashboard.

![](imgs/40.png)

#### Creating an application plan for our API

We need to create an application plan for our users api. Click on `Dashboard` menu and then on `i-users-api`

![](imgs/41.png)

Now, click on `Create Application Plan`.

![](imgs/42.png)

For the `Name` field use: `Basic Plan`. And for the `System name`: `basic-plan`. Now click on `Create Application Plan`.

![](imgs/43.png)

We need to publish our application plan. To do that, click on `Publish`

![](imgs/44.png)

#### Creating an application for our API

