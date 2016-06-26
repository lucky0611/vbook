import os
import random
from flask import Flask, render_template, redirect, json, request,session, url_for, jsonify, send_from_directory
from flask.ext.mysql import MySQL
from werkzeug import generate_password_hash, check_password_hash, secure_filename
from flask.ext.mail import Mail, Message



mysql = MySQL()
app = Flask(__name__)
app.config.from_object(__name__)
app.secret_key = 'blogit'


#email configurations
app.config['MAIL_SERVER']='smtp.gmail.com'
app.config['MAIL_PORT'] = 465
app.config['MAIL_USERNAME'] = 'tiwari2010iit@gmail.com'
app.config['MAIL_PASSWORD'] = 'lucky@611'
app.config['MAIL_USE_TLS'] = False
app.config['MAIL_USE_SSL'] = True
app.config.from_object(__name__)
mail = Mail(app)


#Image folder configurations
app.config['UPLOAD_FOLDER'] = '../blogapp/uploads/'
app.config['ALLOWED_EXTENSIONS'] = set(['txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif'])


# MySQL configurations
app.config['MYSQL_DATABASE_USER'] = 'root'
app.config['MYSQL_DATABASE_PASSWORD'] = ''
app.config['MYSQL_DATABASE_DB'] = 'vbook'
app.config['MYSQL_DATABASE_HOST'] = 'localhost'
mysql.init_app(app)

abc = {'b':['a','b','c']}

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1] in app.config['ALLOWED_EXTENSIONS']


@app.route('/')
def index():
        return render_template('index.html')

#@app.route("/<msg>")
#def main(msg = None):
#        return render_template('index.html', msg = msg)

@app.route('/checkmail', methods=['POST'])
def checkmail():
        try:
                if request.method == 'POST':
                        con = mysql.connect()
                        cursor = con.cursor()
                        cursor.execute("""SELECT user_email,user_username FROM user_signpp""")
                        con.commit()
                        data = cursor.fetchall()
                        if len(data)>0:

                            return jsonify(data= data)
                        else:
                            return ''
                        cursor.close()
                        con.close()
        except Exception as e:
                return render_template('error.html', error = str(e))



@app.route('/checkpageavailability', methods=['POST'])
def checkpage():
        try:
                if request.method == 'POST':
                        con = mysql.connect()
                        cursor = con.cursor()
                        cursor.execute("""SELECT pagename FROM userpages""")
                        con.commit()
                        data = cursor.fetchall()
                        if len(data)>0:

                            return jsonify(data= data)
                        else:
                            return ''
                        cursor.close()
                        con.close()
                else:
                        return 'okk'
                        
        except Exception as e:
                return render_template('error.html', error = str(e))

@app.route('/createpages', methods=['POST'])
def createpage():
        try:
                if session.get('user'):
                        _userid = session.get('user')
                        _pagename = request.form["pgname"]
                        if _pagename:
                            con = mysql.connect()
                            cursor = con.cursor()
                            cursor.execute("""INSERT INTO 
                                            userpages (
                                                pagename,
                                                user_id)
                                            VALUES(%s,%s)""",(_pagename,_userid))
                            con.commit()
                            save_path = '../blogapp/templates/usertemp/'

                            completeName = os.path.join(save_path, _pagename + ".html")         

                            file1 = open(completeName, "w")

                            toFile = """<html>
<head>
    <link href='https://fonts.googleapis.com/css?family=Lato:400,700,300,100' rel='stylesheet' type='text/css'>
   
    <link href="static/css/bootstrap.min.css" rel="stylesheet">
    <!-- Bootstrap theme -->
    <link href="static/css/bootstrap-theme.min.css" rel="stylesheet">
    <script src="static/js/jquery-1.12.1.min.js"></script>
  <script src="static/js/bootstrap.min.js"></script>
<Title>{% if pagetitle %}{{ pagetitle }}{% else %}vbook{% endif %}</Title>
<script>
function showhide(id,id2) {
    var e = document.getElementById(id);
    var f = document.getElementById(id2);
    e.style.display = (e.style.display == 'block') ? 'none' : 'block';
    f.style.display = (f.style.display == 'none') ? 'block' : 'none';
 }
</script>
</head>
<body style="background-color:#f5f5f5">
<div class="Container">
<div class="col-md-12" style="margin-top:30px"> 
<div align="center">
<p style="font-family:lato;font-size:60px" class = "head" >{% if pagetitle %}{{ pagetitle }}{% else %}vbook{% endif %}</p>
<p style="font-family:lato;font-size:20px">{% if abttitle %}{{ abttitle }}{% else %}It's Nice to meet you all{% endif %}</p>
<hr>
<p><span onclick = "showhide('blog','abt')">BLOG</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span onclick = "showhide('abt','blog')">ABOUT ME</span></p>
</div>
</div>
<div id="blog" Align="center" class="col-md-12" style="background-color:#ffffff">
</div>
<div id="abt" class="col-md-12" style="display:none;background-color:#ffffff">
    <br />
    <div class="col-md-5" align="center">

        <image id="mydp"  width="180px" height="180px" />
        <br />
        <p id="myname" style="font-size:18px"></p>
        <p id="mynumber" style="font-size:16px"></p>
        <p id="myemail" style="font-size:16px"></p>
    </div>
    <div align="center" class="col-md-7">
        <p style="font-size:19px"> What I am...</p>
        <p id="abtme" style="font-size:16px"></p>
    </div>
</div>



</div>
<script>
$(function(){
    $.ajax({
    url: '/about',
    type: 'GET',
    success: function(res){
        $("#myname").html(res.data[0][4] + "  " + res.data[0][5]);
        
        $("#mynumber").html("Phone  " + res.data[0][6]);
        $("#myemail").html("Email  " + res.data[0][2]);
        $("#abtme").html(res.data[0][7]);
        $("#mydp").attr('src','../uploads/' + res.data[0][8] + '');
    }
        
    
})

});
$(function(){
$.ajax({
    url: '/getblog',
    type: 'GET',
    success: function(res){
        for (var i = res.data.length - 1; i >= 0; i--) {
            $("#blog")
            .append($("<br />"))
            .append($('<b>')
                .append($('<a>')
                    .attr('href', '/'+ res.data[i].pgname +'/'+ res.data[i].id + '')
                    .append($("<p>")
                        .css("font-size",20)
                        .text(res.data[i].title))))
            .append($('<p>')
                .css("font-size",14)
                .text(res.data[i].date))
            .append($('<p>')
                .css("font-size",18)
                .text(res.data[i].content))
            .append($('<hr>'))
        };

        
        
    }
})
});
</script>
</body>
</html>"""

                            file1.write(toFile)

                            file1.close()
                            return redirect("/userhome")
                            cursor.close()
                            con.close()
                else:
                        return redirect("/")

        except Exception as e:
                return render_template('error.html', error= str(e))


@app.route('/createurcorner')
def corner():
        if session.get('user'):
                return render_template('landingpage.html', msg = session.get('msg'), succ='Account Successfully Activated')

        else:
                return render_template('error.html',error ='Unauthorized request')
       


@app.route('/signup', methods=['POST', 'GET'])
def signUp():
        try:
                if request.method == 'POST':
                        _username = request.form["user"]
                        _email = request.form["email"]
                        _password = request.form["pass"]
                        #msg = Message('Vbook Account Activation', sender='tiwari2010iit@gmail.com', recipients=[_email])
                        #msg.body = "click the link below to activate your account, http://127.0.0.1:5003/landing?user=_username&amp;email=_email&amp;pass=_password/"
                        #mail.send(msg)
                        abc[_email] = [_username,_email,_password]
                        #return redirect('/')
                        #only for testing purposes
                        return redirect("/landing?user=" + _username + "&email=" + _email + "&pass=" + _password)
                        #return redirect(url_for('land', user=_username,email=_email,pass=_password))
                        
                        ##
        except Exception as e:
                return render_template('error.html', error = str(e))

@app.route('/landing', methods=['GET','POST'])
def land():
        
        
        try:
                if request.method == 'GET':
                        _username = request.args.get("user")
                        _email = request.args.get("email")
                        _password = request.args.get("pass")
                        _fname = "N/A"
                        _lname = "N/A"
                        _ph = 1234567890
                        _abt = "I'm a vbooker"
                        _profpic = "468.png"       
                        conn = mysql.connect()
                        cursor = conn.cursor()
                        if _username and _email and _password:
                                if abc[_email] == [_username,_email,_password]:

                                
                                    cursor.execute(
                                        """INSERT INTO
                                    user_signpp (
                                            user_username,
                                            user_email,
                                            user_password,
                                            user_fname,
                                            user_lname,
                                            user_ph,
                                            user_abt,
                                            profilepic)
                                    VALUES (%s,%s,%s,%s,%s,%s,%s,%s)""",(_username,_email,_password,_fname,_lname,_ph,_abt,_profpic))
                                    #del dict[_email]
                                    cursor.execute("SELECT * FROM user_signpp WHERE user_email = '" + _email +"'")
                                    conn.commit()
                                    data = cursor.fetchall()
                                    if len(data) > 0:
                                
                                        if str(data[0][3]) == _password:
                                        
                                            session['user'] = data[0][0]
                                            session['msg'] = list(data[0]) 

        
                                            return redirect('/createurcorner')
                                        
                                        
                                        else:
                                            return render_template('index.html',error="data match not found")
                                    else:
                                        return redirect(url_for('main',msg = 'data missing, try to signup again'))
                                        cursor.close()
                                        conn.close()
                                else:
                                    return "please don't try to hack me..."
                        else:
                                return render_template('error.html', error = "Try to signup again with authentic data")            
        except Exception as e:


                return render_template('error.html', error= str(e))



@app.route('/signin', methods=['GET','POST'])
def signIn():
        con = mysql.connect()
        cursor = con.cursor()
        try:
                
                _user = request.form["usernam"]
                _pass = request.form["passwrd"]
                if _user and _pass:
                        
                        cursor.execute("SELECT * FROM user_signpp WHERE user_username = '" + _user +"'")

                        data = cursor.fetchall()

                        
                        #print 555
                        if len(data) > 0:
                                
                                if str(data[0][3]) == _pass:
                                        
                                        session['user'] = data[0][0]
                                        _uid = data[0][0]
                                        session['msg'] = list(data[0]) 
                                        cursor.execute("""SELECT pagename FROM userpages WHERE user_id =%s""",(_uid))
                                        datap = cursor.fetchall()
                                        
                                        if len(datap) > 0:

                                                return redirect(url_for('userhome'))
                                        else:
                                                return redirect('/createurcorner')
                                        
                                        
                                else:
                                        return render_template('index.html',error="wrong username & Password")
                else:
                        return render_template('index.html', error ="fill the required credintials")
                        
                        
                        

        except Exception as e:
                return render_template('error.html', error = str(e))
        
        #cursor.close()
        #con.close()

@app.route('/logout')
def logout():
        session.pop('user',None)
        session.pop('msg', None)
        session.pop('success', None)
        return redirect('/')


        
@app.route('/addupdate',methods=['POST'])
def addupdate():
    try:
        if session.get('user'):
            _title = request.form['title']
            _description = request.form['textpost']
            _user = session.get('user')
            

            if _title and _description and _user:
                    conn = mysql.connect()
                    cursor = conn.cursor()
                    
                    cursor.execute("""INSERT INTO addpost(post_title, post_content, user_id)
                                        VALUES(%s,%s,%s)""",(_title,_description,_user))
                    conn.commit()
                    cursor.close()
                    conn.close()
                    session['success'] = "Successfully published"
                    return redirect(url_for('userhome'))
                    
                          
            else:
                    return render_template('error.html',error = 'Input error')
        else:
            return render_template('error.html',error = 'Unauthorized input')        
    except Exception as e:
        return render_template('error.html',error = str(e))

@app.route('/userhome')
def userhome():
        if session.get('user'):
                if session.get('success'):
                        return render_template('name.html', msg = session.get('msg'), success = session.get('success'))
                else:
                        return render_template('name.html', msg = session.get('msg'))

        else:
                return render_template('error.html',error ='Unauthorized request')

    

@app.route('/getposts')
def getposts():
        try:
                if session.get('user'):
                        _userid = session.get('user')
                        con = mysql.connect()
                        cursor = con.cursor()
                        cursor.execute("SELECT * FROM addpost WHERE user_id = '" + str(_userid) + "'")
                        posts = cursor.fetchall()
                        posts_dict = []
                        for post in posts:
                            post_dict = {
                            'id': post[0],
                            'title': post[1], 
                            'content': post[2], 
                            'date': post[3]
                            
                            }
                        
                            posts_dict.insert(0, post_dict)
                        return jsonify(data = posts_dict)    

                        cursor.close()
                  
                        con.close()
                else:
                        return redirect('/')
        except Exception as e:
                return render_template('error.html', error = str(e))


@app.route('/deletepost', methods = ['POST', 'GET'])
def deletepost():
        try:
                if session.get('user'):

                        _userid = session.get('user')
                        _postid = request.args.get('id')
                        con = mysql.connect()
                        cursor = con.cursor()
                        cursor.execute("DELETE FROM addpost WHERE user_id = '" + str(_userid) + "' and Id = '" + str(_postid) + "'")
                        con.commit()
                        session.pop('success', None)
                        return redirect('/userhome')

                        cursor.close()
                        con.close()
                else:
                        return redirect('/')
        except Exception as e:
                return render_template('error.html', error = str(e))


   

@app.route('/updatepost', methods=['POST', 'GET'])
def updatepost():
        try:
            if session.get('user'): 
                        _posttitle = request.form["postt"]
                        _postcontent = request.form["postd"]
                        _postid = request.form['postid']
                        con = mysql.connect()
                        cursor = con.cursor()
                        #cursor.execute("UPDATE addpost SET post_title='apple', post_content ='a fruit' WHERE Id = '" + str(_postiid) + "'")
                        cursor.execute("""UPDATE addpost SET post_title= %s, post_content =%s WHERE Id =%s""", (_posttitle, _postcontent, _postid))
            
                        con.commit()
                        
                        return redirect('/userhome')
                        cursor.close()
                        con.close()
            else:
                    return redirect('/')
        except Exception as e:
            return render_template('error.html', error = str(e)) 

        
@app.route('/userprofile', methods = ['GET','POST'])
def userprofile():
        try:
            if session.get('user'):
                        
                        return render_template('profile.html', msg = session.get('msg'))
            else:
                        return redirect('/')
        except Exception as e:
            return render_template('error.html', error = str(e)) 


@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
        if request.method == 'POST':
            file = request.files['file']
            if file and allowed_file(file.filename):
                filename = str(random.randint(99999999,1000000000000000)) + secure_filename(file.filename)
                _userid = session.get('user')
                file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
                con = mysql.connect()
                cursor = con.cursor()
                cursor.execute("""UPDATE user_signpp SET profilepic = %s WHERE Id =%s""", (filename, _userid))
                con.commit()
                cursor.close()
                con.close()
                return ''
                
        return "ooooooppppppssss"

@app.route('/uploads/<filename>')
def uploaded_file(filename):
        return send_from_directory(app.config['UPLOAD_FOLDER'],filename)         

                
@app.route('/getprof', methods=['GET', 'POST'])
def getprof():
        try:

                if session.get('user'):

                    _userid = session.get('user')
                    con = mysql.connect()
                    cursor = con.cursor()
                    cursor.execute("""SELECT profilepic FROM user_signpp WHERE Id = %s""",(_userid))
                    con.commit()
                    data = cursor.fetchall()
                    cursor.close()
                    con.close()
                    return data[0][0]
            
                else:
                    return redirect('/')   
        except Exception as e:
                return render_template("error.html", error = str(e))

@app.route('/updatecprof', methods=['POST'])
def updatecprof():
        try:
                if session.get('user'):
                    _userid = session.get('user')
                    _username = request.form['upduser']
                    _email = request.form['updmail']
                    _phone = request.form['updph']
                    if _userid or _username or _email or _phone:
                            con = mysql.connect()
                            cursor = con.cursor()
                            cursor.execute("""UPDATE user_signpp SET user_username = %s, user_email = %s, user_ph = %s WHERE Id =%s""",(_username,_email,_phone,_userid))
                            cursor.execute("""SELECT * FROM user_signpp WHERE Id = %s""",(_userid))
                            con.commit()
                            datanew = cursor.fetchall()
                            session['msg'] = list(datanew[0])
                            cursor.close()
                            con.close()
                            return redirect('/userprofile')
                            

                else:
                    return redirect('/')
        except Exception as e:
                return render_template('error.html', error = str(e))


@app.route('/updategprof', methods=['POST'])
def updategprof():
        try:
                if session.get('user'):
                    _userid = session.get('user')
                    _fname = request.form['updfname']
                    _lname = request.form['updlname']
                    
                    if _fname or _lname :
                            con = mysql.connect()
                            cursor = con.cursor()
                            cursor.execute("""UPDATE user_signpp SET user_fname = %s, user_lname = %s WHERE Id =%s""",(_fname,_lname,_userid))
                            
                            cursor.execute("""SELECT * FROM user_signpp WHERE Id = %s""",(_userid))
                            con.commit()
                            datanew = cursor.fetchall()
                            session['msg'] = list(datanew[0])
                            cursor.close()
                            con.close()
                            return redirect('/userprofile')
                else:
                    return redirect('/')
        except Exception as e:
                return render_template('error.html', error = str(e))



@app.route('/updateabbprof', methods=['POST'])
def updateabbprof():
        try:
                if session.get('user'):
                    _userid = session.get('user')
                    _abtme = request.form['updabt']
                    if _abtme:
                            con = mysql.connect()
                            cursor = con.cursor()
                            cursor.execute("""UPDATE user_signpp SET user_abt = %s WHERE Id =%s""",(_abtme,_userid))
                            cursor.execute("""SELECT * FROM user_signpp WHERE Id = %s""",(_userid))
                            con.commit()
                            datanew = cursor.fetchall()
                            session['msg'] = list(datanew[0])
                            cursor.close()
                            con.close()
                            return redirect('/userprofile')
                            

                else:
                    return redirect('/')
        except Exception as e:
                return render_template('error.html', error = str(e))




@app.route('/<name>')
def index0(name):
        con = mysql.connect()
        cursor = con.cursor()
        cursor.execute("""SELECT * FROM userpages WHERE pagename = %s""",(name))
        data = cursor.fetchall()

        session['apple'] = data[0][2]
        return render_template('usertemp/' + name +'.html')
        cursor.close()
        con.close()

@app.route('/about')
def about():
        _id = session.get('apple')
        con = mysql.connect()
        cursor = con.cursor()
        cursor.execute("""SELECT * FROM user_signpp WHERE Id = %s""",(_id))
        data = cursor.fetchall()
        return jsonify(data = data)
        cursor.close()
        con.close()


@app.route('/getblog')
def getblog():
        _id = session.get('apple')
        con = mysql.connect()
        cursor = con.cursor()
        cursor.execute("""SELECT * FROM addpost WHERE user_id = %s """,(_id))
        posts = cursor.fetchall()
        cursor.execute("""SELECT * FROM userpages WHERE user_id = %s """,(_id))
        data = cursor.fetchall()
        posts_dict = []
        for post in posts:
            post_dict = {
            'id': post[0],
            'title': post[1], 
            'content': post[2], 
            'date': post[3],
            'pgname': data[0][1]
                            
            }
                        
            posts_dict.insert(0, post_dict)
        return jsonify(data = posts_dict)  
        cursor.close()
        con.close()

@app.route('/<pagename>/<postid>')
def getfull(pagename,postid):
        con = mysql.connect()
        cursor = con.cursor()
        cursor.execute("""SELECT * FROM addpost WHERE Id = %s""",(postid))
        data = cursor.fetchall()
        _id = data[0][4]
        cursor.execute("""SELECT * FROM user_signpp WHERE Id = %s""",(_id))
        dataa = cursor.fetchall()
        cursor.execute("""SELECT * FROM userpages WHERE user_id = %s""",(_id))
        dataaa = cursor.fetchall()
        return render_template('usertemp/posttemp.html',msg=data, abt = dataa, blog= pagename)
        cursor.close()
        con.close()









if __name__ == "__main__":
        
    app.debug = True
    app.run(port=5003)	
