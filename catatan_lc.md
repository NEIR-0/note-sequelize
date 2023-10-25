# JSON
1. foreignkey harus PASCAL CASE!

# INSTALATION
1. npm init -y
2. npm i express ejs
3. npm install --save sequelize
4. npm install --save pg pg-hstore # Postgres

# MIGRATE
1. npm install --save-dev sequelize-cli
2. npx sequelize-cli init
- CONFIG:
"development": {
    "username": "postgres",
    "password": "postgres",
    "database": "bookstore-simulasi-lc",
    "host": "localhost",
    "dialect": "postgres"
},

3. npx sequelize-cli db:create
4. npx sequelize-cli model:generate --name Book --attributes name:string,genre:string,AuthorId
    - name gak usah pake "s" (Book), Fk harus pascal case "AuthorId"
4. npx sequelize-cli db:migrate
- MIGRATE:
    AuthorId: {
        type: Sequelize.INTEGER,
        references: {
          model: 'Authors',
          key: 'id'
        },
        onUpdate: 'cascade',
        onDelete: 'cascade'
      },

!! OPTIONAL !! ===> SEKELETON (BIASANYA BUAT FK)
- SEKELETON:
    1. npx sequelize-cli migration:generate --name add-konteks
        - up:
            await queryInterface.addColumn('nama_tabel', 'nama_key', {  <<<<<<<<<< (pake "await", ubah "DataTypes" jadi "Sequelize")
                type: Sequelize.STRING,
                        references: {
                        model: 'Authors',   <<<<<<<<<< (nama tabelnya!) 
                        key: 'id'
                    },
                    onUpdate: 'cascade',
                    onDelete: 'cascade'
            });   

        - down:
            await queryInterface.removeColumn('nama_tabel', 'nama_key', { /* query options */ });   <<<<<<<<<< (pake "await")
    
    2. npx sequelize-cli db:migrate
    
- MODEL:
    - static associate(models):
        Author.hasMany(models.Book);    <<<<<< (pk)
        Book.belongsTo(models.Author);  <<<<<< (fk)

    - validate:
    price: {
      type: DataTypes.INTEGER,
      allowNull: false,
      validate: {
        notNull: {
          args: true,
          msg: "price gak boleh null"
        },
        notEmpty: {
          args: true,
          msg: "price gak boleh Empty"
        },
        notZero(value) {
          if (value < 0) { // bisa pke "this", sesuai key di init (anngap aja models biasa)
            throw new Error('stock minimal 0');
          }
        } 
      }
    },
    
    - hooks:
    Book.addHook('beforeCreate', (instance, options) => {
        /*
        title = 'Payung Kumbuh'
        isbn = '123123'
        ==========================
        isbn = payung_kumbuh123123
        */
        const formating = instance.title.split(" ").join("_")
        instance.isbn = formating+instance.isbn
    });

    !! OPTIONAL !! ===> SEKELETON (Models)
    - SEKELETON:
         Book.init({
            title: DataTypes.STRING,
            isbn: DataTypes.STRING,
            price: DataTypes.INTEGER,
            stock: DataTypes.INTEGER,

            // foreign key
            AuthorId: DataTypes.INTEGER,    <<<<<<<<< (tambahin foreign key di models sebelum seedeing)
        }

# SEEDING
1. npx sequelize-cli seed:generate --name add-konteksnya
- SEEDER:
    - import:
        const fs = require("fs")
    - up:
        const data = JSON.parse(fs.readFileSync("./authors.json", "utf-8"))
        data.forEach(el => {
            delete el.id
            el.createdAt = new Date()
            el.updatedAt = new Date()
        });
        await queryInterface.bulkInsert("Authors", data)    <<<<<<<< (pake "await" bukan return)
    - down:
        await queryInterface.bulkDelete('Authors', null, {});   <<<<<<<< (pake "await" bukan return)

# AFTER SET-UP
1. mkdir Controllers Views Routes
    - Helper (option)
2. touch app.js .gitignore Controllers/controller.js Views/home.ejs Routes/index.js
    - Helper/helper.js (option)

# APP.JS
- EXPRESS:
const express = require('express')
const app = express()
const port = 3000
const index = require("./Routes/index")     <<<<<<<<<< (router!!)

app.set('view engine', 'ejs')       <<<<<<<<<< ("pug" ubah jadi "ejs")
app.use(express.urlencoded({extended: true}))

app.use("/", index)     <<<<<<<<<<<< (pake "app.use")

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`)
})

# .GITIGNORE
node_modules

# ROUTES/INDEX.JS
const express = require('express')
const routes = express.Router()     <<<<<<<<<< (ubah jadi "routes = express.Router()")
const authors = require("./authors")    <<<<<<<<<< (file baru)
const books = require("./books")    <<<<<<<<<< (file baru)

routes.use("/", authors)    <<<<<<<<<< (pake "router.use")
routes.use("/", books)  <<<<<<<<<< (pake "router.use")

module.exports = routes

# ROUTES/BOOKS.JS (CONTOH!)
const express = require('express')
const routes = express.Router()     <<<<<<<<<< (ubah jadi "routes = express.Router()")
const Controllers = require("../Controllers/controller")    <<<<<<<<<< (panggil Controllers)

// home
routes.get("/books", Controllers.home)

// add & post (urlnya sama, url gak pake titik INGAT!)
routes.get("/books/add", Controllers.formBooks)     <<<<<<<<<< (pake "routes.get")
routes.post("/books/add", Controllers.postBooks)    <<<<<<<<<< (pake "routes.post")

module.exports = routes     <<<<<<<<<< (jangan lupa "module.export"!)

# CONTROLLERS/CONTROLLERS.JS
- IMPORT:
    const {Book, Author} = require("../models")     <<<<<<<<<< (panggil modelnya!, {Book, Author} ini adalah nama class dalam filenya!)
    const { Op } = require("sequelize");    <<<<<<<<<< (buat "where" dan "kawan2")
    const {formating} = require("../Helper/helper")     <<<<<<<<<< (helper optional)

- DEFAULTNYA:
    - FINDALL / FINDONE:
        static async showAuthors(req, res){
            try {
                const data = await Book.findAll({
                    where:{
                        stock: {
                            [Op.gt]: 0  <<<<<<<< (greter > 0)
                        }
                    },
                    include:{
                        model: Author   <<<<<<<< (ini nama model, karena gak pake "s")
                    },
                    where: {
                        '$Author.name$': name, // this interisting. '$Author.name$', lokasi data nya!!
                    },
                    order:[ ['id', 'ASC'] ]     <<<<<<< (kalo order pake 2 array)
                })
            } catch (error) {
                res.send(error)
            }
        }
    -  CREATE (ADD, DIBAGIAN POST):
        1. DEFAULT:
            const {Title, ISBN, Stock, Price, Author} = req.body    <<<<<<<< (dapet dari "formAdd")
            await Book.create({ title: Title, isbn: ISBN, price: Price, stock: Stock, AuthorId: Author });  <<<<<<<< (pake ".create")
            res.redirect("/books/")     <<<<<<< (redirect)
            
    -  UPDATE
        const {id} = req.params
        const {restock} = req.body
        await Book.update({ stock: restock }, {     <<<<<<<< (pake ".update")
            where: {
                id: id
            }
        });
        res.redirect("/books/emptyList")    <<<<<<< (redirect)

    -  DELETE
        const {id} = req.params
        await Book.destroy({    <<<<<<<< (pake ".destroy")
            where: {
                id: id
            }
            });
        res.redirect("/books/")     <<<<<<< (redirect)
    
    - VALIDATE ERROR:
        1. POST (ADD/UPDATE):
            // catch
            } catch (error) {
            if(error.name === "SequelizeValidationError"){      <<<<<<< (error dari validasi bagian model "msg")
                let arrError = []
                error.errors.forEach(el => {
                    arrError.push(el.message)
                });

                res.redirect(`/books/add?error=${arrError}`)    <<<<<<< (redirect, kebagian "form")
            }
            else{
                res.send(error)
            }

        2. FORM (ADD/UPDATE):
            // validate
            const {error} = req.query   <<<<<<< (ambil dari query)
            const data = await Author.findAll()

            if(error){  <<<<<<< (perkondisian "(name)" = truthty)
                const arrError = error.split(",")   <<<<<<< (kalo ada error di 'error.split(",")', biar jadi array)
                res.render("formBooks", {
                    data: data,
                    error: arrError <<<<<<< (masukin yang udah jadi array (arrError))
                })
            }
            else{   <<<<<<< (perkondisian "else")
                res.render("formBooks", {
                    data: data,
                    error: []   <<<<<<< (masukin error array kosong, "error: []")
                })
            }

        2. EJS (FORM):
            <!-- validate -->
            <% if (error.length > 0) { %>   <<<<<<< (pake kondisi dulu, "if")
                <ul>
                    <% error.forEach(el => { %>     <<<<<<< (baru pake "looping")
                        <li style="color: red;"><%= el %></li>
                    <% }) %>
                </ul>
            <% } %>

    - FILTERING:
        !! PERINGATAN !! ==> KALO LU PUNYA 2 FILTERING (BUTTON/FORM) HARUS DI PISAH
        1. EJS:
            <!-- filter -->
            <a href="/books/price?price=20000">     <<<<<<< (button, linknya ke url baru "/books/price?price=20000" dan langsung di hardcode querynya! "?price=20000")
            <button>> Rp. 20.000</button>
            </a>
            <form action="/books?name=" method="get">   <<<<<<< (form method "get", linknya ke url yang sama '/books?name="', cuman dikasih query doang "?name=". AUTO DAPET VALUE (BASE ON INPUTS) PAS SUBMIT DI TEKAN)
            <select name="name">
                <option value="" <%= name === "" ? "selected" : "" %>>Select</option>   <<<<<<< (validasi buat "selected")
                <option value="J. K. Rowling" <%= name === "J. K. Rowling" ? "selected" : "" %>>J. K. Rowling</option>
                <option value="George R. R. Martin" <%= name === "George R. R. Martin" ? "selected" : "" %>>George R. R. Martin</option>
                <option value="Suzanne Collins" <%= name === "Suzanne Collins" ? "selected" : "" %>>Suzanne Collins</option>
                <option value="James Dashner"  <%= name === "James Dashner" ? "selected" : "" %>>James Dashner</option>
            </select>
            <button type="">Filter</button>
            </form>

        2. CONTROLLERS/CONTROLLER.JS:
            - FORM:
                const {name} = req.query    <<<<<<< (querynya! dari "?name=")
                let data;   <<<<<<< (var baru)
                if(name){
                    data = await Book.findAll({
                        where:{
                            stock: {
                                [Op.gt]: 0
                            }
                        },
                        include:{
                            model: Author
                        },
                        where: {    <<<<<<< (kalo filternya di dalem include bisa pake "where" ini)
                            '$Author.name$': name,  <<<<<<< (this interisting. '$Author.name$', lokasi data nya!!)
                        },
                        order:[
                            ['id', 'ASC']
                        ]
                    })
                }
                else{
                    data = await Book.findAll({
                        where:{
                            stock: {
                                [Op.gt]: 0
                            }
                        },
                        include:{
                            model: Author
                        },
                        order:[
                            ['id', 'ASC']
                        ]
                    })
                }
                
                formating(data)     <<<<<<< (helper)

                if(name){   <<<<<<< (true)
                    res.render("showBooks", {   <<<<<<< (redirect)
                        data: data,
                        name: name  <<<<<<< (masukin querynya, buat perkodnsiian nambah "selected")
                    })
                }

                else{   <<<<<<< (false)
                    res.render("showBooks", {
                        data: data,
                        name: ""    <<<<<<< (masukin string kosong aja "", biar gak error!)
                    })
                }

            - BUTTON:
                const {price} = req.query   <<<<<<< (ambil query nya!)
                let data;   <<<<<<< (var baru)
                if(price){
                    data = await Book.findAll({
                        where:{
                            stock: {
                                [Op.gt]: 0
                            },
                            price:{
                                [Op.gt]: Number(price)
                            }
                        },
                        include:{
                            model: Author
                        },
                        order:[
                            ['id', 'ASC']
                        ]
                    })
                }
                else{
                    data = await Book.findAll({
                        where:{
                            stock: {
                                [Op.gt]: 0
                            }
                        },
                        include:{
                            model: Author
                        },
                        order:[
                            ['id', 'ASC']
                        ]
                    })
                }

                formating(data)     <<<<<<< (helper)

                res.render("showBooks", {   <<<<<<< (url sama kayak form)
                    data: data,
                    name: ""    <<<<<<< (biar gak eror dan bisa perkodnsiian nambah "selected")
                })


<!-- CATATAN TAMBAHAN -->
# decrement / increment
await Book.decrement('stock', {  <<<<<<< ("stock" nama colum / key, "Book.decrement / Book.increment")
    by: 1,  <<<<<<< (berapa banyak yang dikurangi (disini gw mau "1")) 
    where: {
        id: id
    }
});

# REQ
1. req.params
2. req.query
3. req.body

# INPUT RADIO
1. ejs:
    <form action="/books/radio" method="post">
        <h1><%= namaAuthors %></h1>
        <ul>
            <% data.forEach(el => { %>
                <li>
                    <input type="radio" id="test" name="namaAuthors" value="<%= el.name %>" <%= el.name === namaAuthors ? "checked" : "" %>>
                    <label for="test"><%= el.name %></label>
                </li>
            <% }) %>
        </ul>

        <button type="submit">submit</button>
    </form>

2. controller post:
    const {namaAuthors} = req.body      <<<<<<<< (dari "name" input radio)
    const data = await Author.findAll()

    if(namaAuthors){
        res.render("testRadio", {
            data: data,
            namaAuthors: namaAuthors    <<<<<<<< (masukin "namaAuthors")
        })
    }
    else{
        res.render("testRadio", {
            data: data,
            namaAuthors: ""     <<<<<<<< (musukin "namaAuthors" jadi "", biar gak error!)
        })
    }


!! JANGAN LUPA !!
1. TRY & CATCH
2. KALO GAK ADA YANG MUNCUL SAAT "RES.SEND(DATA)" COBA PAKAI "CONSOLE.LOG(DATA)"
