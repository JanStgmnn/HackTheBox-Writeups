# Cyber Santa is Coming to Town: Toy Management

![Author: crxzii](https://img.shields.io/badge/Author-crxzii-blue.svg) ![Contest Date: 05.12.2021](https://img.shields.io/badge/Contest%20Date-05.12.2021-lightgrey.svg)

![Solve Moment: During The Contest](https://img.shields.io/badge/Solve%20Moment-During%20The%20Contest-brightgreen.svg) ![Category: Web](https://img.shields.io/badge/Category-Web-lightgrey.svg) ![Score: 300](https://img.shields.io/badge/Score-300-brightgreen.svg)

## Description

> The evil elves have changed the admin access to Santa's Toy Management Portal. Can you get the access back and save the Christmas?


## Attached Files

- web_toy_management.zip

A Zip file containing the full docker source code.

## Summary

We use a simple SQL injection to get admin access and read the database containing the flag.

## Flag

```
HTB{1nj3cti0n_1s_in3v1t4bl3}
```

## Detailed Solution

We are greeted with a login window, with no idea what to do.

![Website Login Window](https://i.gyazo.com/89203c349c146c3f7dee08ec7071643f.png)

To get a better understanding of the website, we look at the demo files.<br>
Looking at the demo files, we can see a `database.db`. We can see, that the flag is loaded into the toylist table:

```sql
--
-- Dumping data for table `toylist`
--

INSERT INTO `toylist` (`id`, `toy`, `receiver`, `location`, `approved`) VALUES
(1,  'She-Ra, Princess of Power', 'Elaina Love', 'Houston', 1),
(2, 'Bayblade Burst Evolution', 'Jarrett Pace', 'Dallas', 1),
(3, 'Barbie Dreamhouse Playset', 'Kristin Vang', 'Austin', 1),
(4, 'StarWars Action Figures', 'Jaslyn Huerta', 'Amarillo', 1),
(5, 'Hot Wheels: Volkswagen Beach Bomb', 'Eric Cameron', 'San Antonio', 1),
(6, 'Polly Pocket dolls', 'Aracely Monroe', 'El Paso', 1),
(7, 'HTB{f4k3_fl4g_f0r_t3st1ng}', 'HTBer', 'HTBland', 0);
-- --------------------------------------------------------
```

The toylist can be accessed through the endpoint `/api/toylist`. But we need to be logged in as admin to access the endpoint.

```js
router.get('/api/toylist', AuthMiddleware, async (req, res) => {
	return db.getUser(req.data.username)
		.then(user => {
			approved = 1;
			if (user[0].username == 'admin') approved = 0;
			return db.listToys(approved)
				.then(toyInfo => {
					return res.json(toyInfo);
				})
				.catch(() => res.status(500).send(response('Something went wrong!')));
		})
		.catch(() => res.status(500).send(response('Something went wrong!')));
});
```

Looking at the `loginUser` function from the database, we can spot something interesting though! We have an SQL injection!

```js
async loginUser(user, pass) {
		return new Promise(async (resolve, reject) => {
			let stmt = `SELECT username FROM users WHERE username = '${user}' and password = '${pass}'`;
			this.connection.query(stmt, (err, result) => {
				if(err)
					reject(err)
				try {
					resolve(JSON.parse(JSON.stringify(result)))
				}
				catch (e) {
					reject(e)
				}
			})
		});
	}
```

Using a simple payload for an SQL injection, we can login as admin:

![SQL Injection Payload](https://i.gyazo.com/33d3525bcafcba7143e12783af3a360b.png)

Being logged in, we can see the toylist loaded on the dashboard and can just read our flag:

![Flag on Dashboard](https://i.gyazo.com/1211b522586ce5dad619c161ca80d021.png)
