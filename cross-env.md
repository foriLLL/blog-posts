https://stackoverflow.com/questions/61927814/how-to-disable-open-browser-in-cra

For Windows you can edit/create a script in package.json like this one:

```
"start": "set BROWSER=none && react-scripts start"
```

For Linux based OS, just delete the "set" and the "&&" and it will work:
```
"start": "BROWSER=none react-scripts start"
```