npm i express mongoose dotenv body-parser multer
model dept.js
let db = require('../db')

let schema = new db.Schema({
    name: {
        type: String,
        required: true
    },
    city: {
        type: String,
        required: true
    }
});

let Dept = db.model('Dept', schema);

module.exports = Dept


deptRouter.js

let Dept = require('../Models/Dept')
const router = require('express').Router()

//get all
router.get('/dept/findall' , async function (req , res){
    let alldept = await Dept.find({})
    return res.json(alldept)
})

// Add New Department
router.post('/dept/addnew' , async function (req,res) {
    let {name , city} = req.body
    let d = new Dept
    d.name = name
    d.city = city
    await d.save()
    return res.json(d)
})

// Update Department
router.post('/dept/update/:id', async function (req, res) {
    let {name, city} = req.body;
    let id = req.params.id;
    let d = await Dept.findById(id);
    d.name = name
    d.city = city
    await d.save()
    return res.json(d);
})

// Delete Department
router.get('/dept/delete/:id', async function (req, res) {
    let id = req.params.id
    let d = await Dept.findByIdAndDelete(id)
    return res.json(d)
})

// Get Single Department
router.get('/dept/getsingle/:id', async function (req, res) {
    let id = req.params.id
    let d = await Dept.findById(id)
    return res.json(d)
})

module.exports = router

db.js
let mongoose = require('mongoose')
mongoose.connect("mongodb://localhost:27017/prc1")
module.exports = mongoose;

index.js
let express = require('express')
let cors = require('cors')

let server = express()
server.use(express.json())
server.use(cors())

let deptRoute = require('./Routes/deptRoute')
server.use(deptRoute)

let empRoute = require('./Routes/empRoute')
server.use(empRoute)

server.listen(2000)


index.php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Departments</title>
</head>
<body>
    <h1>Show All Departments</h1>
    <a href="adddept.php">Add Department</a>

    <br>
    
    <a href="emp.php">View emp mamangemnet</a>
    <br>
    <table border="1">
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>City</th>
                <th>Edit</th>
                <th>Delete</th>
            </tr>
        </thead>
        <tbody>
            <?php
                $curl = curl_init();

                curl_setopt_array($curl, array(
                  CURLOPT_URL => 'localhost:2000/dept/findall',
                  CURLOPT_RETURNTRANSFER => true,
                  CURLOPT_ENCODING => '',
                  CURLOPT_MAXREDIRS => 10,
                  CURLOPT_TIMEOUT => 0,
                  CURLOPT_FOLLOWLOCATION => true,
                  CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
                  CURLOPT_CUSTOMREQUEST => 'GET',
                ));
                
                $response = curl_exec($curl);
                
                curl_close($curl);
                // echo $response;
                $response = json_decode($response, true);

                foreach ($response as $data) 
                {
                    echo "<tr>";
                    echo "<td>$data[_id]</td>";
                    echo "<td>$data[name]</td>";
                    echo "<td>$data[city]</td>";
                    echo "<td><a href='upddept.php?id=$data[_id]'>Update</a></td>";
                    echo "<td><a href='deldept.php?id=$data[_id]'>Delete</a></td>";
                    echo "</tr>";
                }
            ?>
        </tbody>
    </table>
</body>
</html>

adddept.php
<?php
    if(isset($_POST['save']))
    {
        $curl = curl_init();

        curl_setopt_array($curl, array(
        CURLOPT_URL => 'localhost:2000/dept/addnew',
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_ENCODING => '',
        CURLOPT_MAXREDIRS => 10,
        CURLOPT_TIMEOUT => 0,
        CURLOPT_FOLLOWLOCATION => true,
        CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
        CURLOPT_CUSTOMREQUEST => 'POST',
        CURLOPT_POSTFIELDS =>'{
            "name" : "'.$_POST['name'].'",
            "city" : "'.$_POST['city'].'"
        }',
        CURLOPT_HTTPHEADER => array(
            'Content-Type: application/json'
        ),
        ));

        $response = curl_exec($curl);

        curl_close($curl);
        
        header('location: index.php');
    }
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Add Department</title>
</head>
<body>
    <form method="post">
        <input type="text" name="name" id="name" placeholder="Enter Name">
        <input type="text" name="city" id="city" placeholder="Enter City">
        <button type="submit" name="save">save</button>
    </form>
</body>
</html>

deldept.php
<?php
    $curl = curl_init();

    curl_setopt_array($curl, array(
      CURLOPT_URL => 'localhost:2000/dept/delete/' . $_GET['id'],
      CURLOPT_RETURNTRANSFER => true,
      CURLOPT_ENCODING => '',
      CURLOPT_MAXREDIRS => 10,
      CURLOPT_TIMEOUT => 0,
      CURLOPT_FOLLOWLOCATION => true,
      CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
      CURLOPT_CUSTOMREQUEST => 'GET',
    ));
    
    $response = curl_exec($curl);
    
    curl_close($curl);
    header('location: index.php');
?>

upddept.php
<?php

    if(isset($_POST['save']))
    {
        $curl = curl_init();

        curl_setopt_array($curl, array(
        CURLOPT_URL => 'localhost:2000/dept/update/' . $_GET['id'],
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_ENCODING => '',
        CURLOPT_MAXREDIRS => 10,
        CURLOPT_TIMEOUT => 0,
        CURLOPT_FOLLOWLOCATION => true,
        CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
        CURLOPT_CUSTOMREQUEST => 'POST',
        CURLOPT_POSTFIELDS =>'{
            "name" : "'. $_POST['name'] .'",
            "city" : "'. $_POST['city'] .'"
        }',
        CURLOPT_HTTPHEADER => array(
            'Content-Type: application/json'
        ),
        ));

        $response = curl_exec($curl);

        curl_close($curl);
        header('location: index.php');
    }
    else
    {
        $curl = curl_init();
    
        curl_setopt_array($curl, array(
          CURLOPT_URL => 'localhost:2000/dept/getsingle/' . $_GET['id'],
          CURLOPT_RETURNTRANSFER => true,
          CURLOPT_ENCODING => '',
          CURLOPT_MAXREDIRS => 10,
          CURLOPT_TIMEOUT => 0,
          CURLOPT_FOLLOWLOCATION => true,
          CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
          CURLOPT_CUSTOMREQUEST => 'GET',
        ));
        
        $response = curl_exec($curl);
        
        curl_close($curl);
        
        $response = json_decode($response, true);
    }
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Update Department</title>
</head>
<body>
    <form method="post">
        <input type="text" value="<?php echo $response['name']; ?>" name="name" id="name" placeholder="Enter Name">
        <input type="text" value="<?php echo $response['city']; ?>" name="city" id="city" placeholder="Enter City">
        <button type="submit" name="save">save</button>
    </form>
</body>
</html>

#fileupload
#server.js


const express = require('express')
const multer  = require('multer')
const path = require('path');
const app = express();
app.use(express.static('public'));  
app.use(express.urlencoded({extended: true}));
app.listen(8000);

var options=multer.diskStorage(
  {
    destination:(req,file,cb)=>{
      if(file.mimetype!=="image/jpeg")
        return cb("cant")
      cb(null,"./uploads")  
    } ,
    filename: (req,file,cb)=>{
    }
  })
  var options=multer.diskStorage({destination:function(req,file,cb){
    console.log(file)
    if (file.mimetype !== 'image/jpeg') 
      { 
        return cb('Invalid file format'); 
      }
      
      cb(null, './uploads');
  } ,filename:function(req,file,cb){
    console.log(file)
    console.log(path.extname(file.originalname))
    cb(null, (Math.random().toString(30)).
          slice(5, 10) + Date.now() 
          + path.extname(file.originalname));
  }});

  var upload= multer({ storage: options });
  
  app.get("/",function(req,res){
    res.send("<a href='file_upload.html'>Upload your file here</a>")
  })
  
  app.post("/file_upload",upload.single("myfile"),function(req,res){
    res.send("Files are uploaded "+req.body.fname +" "+req.body.lname);
  });
  
  app.post("/photos_upload",upload.array("photos",2),function(req,res){
    res.send("File is uploaded "+req.body.fname+" "+req.body.lname);
  });
  
  app.use(function (err, req, res, next) {
    if (err instanceof multer.MulterError) 
    {
      console.log("ERRRR");
      res.status(500).send("file upload  err "+err.message);
  
    }
    else 
    {
      console.log("ERRRR "+err);
    }
  });


#public/file_upload.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=<device-width>, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>Singe file upload</h1>
    <form method="post" enctype="multipart/form-data" action="file_upload">
        First Name: <input type="text" name="fname"><br>
        Last Name: <input type="text" name="lname"><br>
        File: <input type="file" name="myfile"><br>
        <input type="submit">
    </form>
    <h1>Multi file upload</h1>
    <form method="post" enctype="multipart/form-data" action="photos_upload">
        First Name: <input type="text" name="fname"><br>
        Last Name: <input type="text" name="lname"><br>
        File: <input type="file" name="photos" multiple="multiple"><br>
        <input type="submit">
    </form>
</body>
</html>

login:-
app.use(
  session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: true,
    cookie: { maxAge: 60000 },
  })
);
// Dummy user data
const users = {
  user1: "password1",
  user2: "password2",
};

    app.post("/login", (req, res) => {
  const { username, password } = req.body;
  if (users[username] && users[username] === password) {
    req.session.isLoggedIn = true;
    req.session.username = username;
    res.redirect("/dashboard");
  } else {
    res.send("Invalid username or password! <a href='/'>Try again</a>");
  }
});

app.get("/dashboard", (req, res) => {
  if (req.session.isLoggedIn) {
    res.sendFile(__dirname + "/views/dashboard.html");
  } else {
    res.redirect("/");
  }
});

login.html:-
<form action="/login" method="POST">
    <label for="username">Username:</label>
    <input type="text" id="username" name="username" required><br><br>
    <label for="password">Password:</label>
    <input type="password" id="password" name="password" required><br><br>
    <button type="submit">Login</button>
  </form>