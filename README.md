# Creamos el proyecto

> ng new frontend

* Creamos el componente login

> ng g c auth/login

* Creamos el componente register

> ng g c auth/register

* Creamos el servicio auth

> ng g s services/auth

* Creamos la interface que tiene el formato de la respuesta de nuestro api

> ng g i jwt-response

* Creamos la interface que tiene el formato de nuestro usuario

> ng g i user

* Iniciamos nuestro aplicativo

> ng serve

* Creamos una nueva carpeta quese llame models dentro de app y movemos las dos interfaces anteriormente creadas

# Editamos los archivos de interfaz (modelos)

1. Abrimos nuestra interfaz user.ts

```typescript
    export interface UserI {
    id: number,
    name: string,
    email: string,
    password: string
    }
```
2. Abrimos nuestra interfaz jwt-reponse.ts

```typescript
    export interface JwtResponseI {
    dataUser: {
        id: number,
        name: string,
        email: string,
        accessToken: string,
        expiresIn: string
    }
    }
```

3. Editamos nuestro app.component.ts

```html
<router-outlet></router-outlet>
```

4. Comenzaremo a trabajar las rutas

    1. Generamos desde la consola un modulo aparte para las rutas
    > ng g m auth
    2. Este fichero se devio haber creado en la ruta src/app/auth/auth.module.ts, en esa misma ruta crearemos un nuevo archivo que se llame auth-routing.module.ts
    3. Copiamos lo que teniamos en nuestro archivo general de rutas (app-routing.module.ts) y lo pegamos en el archivo que acabamos de crear
    ```typescript
        import { NgModule } from '@angular/core';
        import { Routes, RouterModule } from '@angular/router';

        const routes: Routes = [];

        @NgModule({
        imports: [RouterModule.forRoot(routes)],
        exports: [RouterModule]
        })
        export class AppRoutingModule { }
    ```
    4. Procedemos a editar nuestro archivo
     ```typescript 
        import { NgModule } from '@angular/core';
        import { Routes, RouterModule } from '@angular/router';
        // Importamos nuestros componentes
        import { LoginComponent } from './login/login.component';
        import { RegisterComponent } from './register/register.component';

        // Creamos las rutas para dichos componentes
        const routes: Routes = [
        { path: 'register', component: RegisterComponent },
        { path: 'login', component: LoginComponent }
        ];

        @NgModule({
        // Modificamos a forChild
        imports: [RouterModule.forChild(routes)],
        exports: [RouterModule]
        })
        // Modificamos al nombre de la clase
        export class AuthRoutingModule { }
    ```
    5. Procedemos a editar nuestro auth.module.ts
    ```typescript 
        import { NgModule } from '@angular/core';
        import { CommonModule } from '@angular/common';
        // Procedemos a importar los modulos necesarios
        import { FormsModule } from '@angular/forms';
        import { HttpClientModule } from '@angular/common/http';
        // Importamos nuestro archivo de rutas
        import { AuthRoutingModule } from './auth-routing.module';
        // Importamos nuestros componentes
        import { RegisterComponent } from './register/register.component';
        import { LoginComponent } from './login/login.component';
        // Importamos nuestro servicios
        import { AuthService } from '../services/auth.service';

        @NgModule({
        declarations: [RegisterComponent, LoginComponent],
        // Colocamos nuestros modulos como imports para poder usarlos en nuestros componentes
        imports: [
            CommonModule,
            FormsModule,
            AuthRoutingModule,
            HttpClientModule
        ],
        // Colocamos nuestro servicio como provider
        providers: [AuthService]

        })
        export class AuthModule { }
    ```
    6. Editamos nuestro archivo app-routing.module.ts
    ```typescript
    import { NgModule } from '@angular/core';
    import { Routes, RouterModule } from '@angular/router';

    const routes: Routes = [
    // Creamos las rutas
    { path: '', redirectTo: '/auth', pathMatch: 'full' },
    // Creamos la ruta que redireccionara a auth
    { path: 'auth', loadChildren: './auth/auth.module#AuthModule' }
    ];

    @NgModule({
    imports: [RouterModule.forRoot(routes)],
    exports: [RouterModule]
    })
    export class AppRoutingModule { }
    ```

# Trabajando nuestros serevices

1. Editamos nuestro archivo auth.service.ts

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
// Importamos nuestra interfaces
import { UserI } from '../models/user';
import { JwtResponseI } from '../models/jwt-response';
// Importamos los operadores
import { tap } from 'rxjs/operators';
import { Observable, BehaviorSubject } from 'rxjs';


@Injectable()
export class AuthService {

    // Creamos nuestras propiedades, la ruta
  AUTH_SERVER: string = 'http://localhost:3000';
  // para guardar el ulimo estado
  authSubject = new BehaviorSubject(false);
  private token: string;
  // Iniciamos el httplient por medio del constructor
  constructor(private httpClient: HttpClient) { }
    // Creamos el metodo de registro
  register(user: UserI): Observable<JwtResponseI> {
    return this.httpClient.post<JwtResponseI>(`${this.AUTH_SERVER}/register`,
      user).pipe(tap(
        (res: JwtResponseI) => {
          if (res) {
            // guardar token
            this.saveToken(res.dataUser.accessToken, res.dataUser.expiresIn);
          }
        })
      );
  }
// Metodo de login
  login(user: UserI): Observable<JwtResponseI> {
    return this.httpClient.post<JwtResponseI>(`${this.AUTH_SERVER}/login`,
    // Este operator segun la documentacion lo que hace es crear un segundo observable que retorna lo que retorna el primero
    // Un pipe se subcribe al primer observable para poder llevar a cabo una accion intermediaria miestras se hace una peticion
      user).pipe(tap(
        (res: JwtResponseI) => {
          if (res) {
            // guardar token
            this.saveToken(res.dataUser.accessToken, res.dataUser.expiresIn);
          }
        })
      );
  }
// Para cerrar sesion
  logout(): void {
    this.token = '';
    localStorage.removeItem("ACCESS_TOKEN");
    localStorage.removeItem("EXPIRES_IN");
  }
// Para guardar el token en el localStorage
  private saveToken(token: string, expiresIn: string): void {
    localStorage.setItem("ACCESS_TOKEN", token);
    localStorage.setItem("EXPIRES_IN", expiresIn);
    this.token = token;
  }
// Para obrtener el token guardado en el local storage
  private getToken(): string {
    if (!this.token) {
      this.token = localStorage.getItem("ACCESS_TOKEN");
    }
    return this.token;
  }

}
```

# Modificamos el componente

1. Abrimos el archivo login.component.html

```html
<div class="container">
  <div class="login">
    <div class="login-triangle"></div>

    <h2 class="login-header">Log in</h2>
    <!-- Modificamos algunas cosas como colocar el evento-->
    <form #frmLogin="ngForm" class="login-container" (ngSubmit)="onLogin(frmLogin)">
      <p>
        <input type="email" name="email" placeholder="Email" ngModel required>
      </p>
      <p>
        <input type="password" name="password" placeholder="Password" ngModel required>
      </p>
      <p>
        <input type="submit" value="Send">
      </p>
    </form>
  </div>
</div>
```

2. Editamos el archivo global de css styles.css
```css
/* 'Open Sans' font from Google Fonts */
@import url(https://fonts.googleapis.com/css?family=Open+Sans:400,700);

body {
  background: #456;
  font-family: 'Open Sans', sans-serif;
}

.login {
  width: 400px;
  margin: 16px auto;
  font-size: 16px;
}

/* Reset top and bottom margins from certain elements */
.login-header,
.login p {
  margin-top: 0;
  margin-bottom: 0;
}

/* The triangle form is achieved by a CSS hack */
.login-triangle {
  width: 0;
  margin-right: auto;
  margin-left: auto;
  border: 12px solid transparent;
  border-bottom-color: #28d;
}

.login-header {
  background: #28d;
  padding: 20px;
  font-size: 1.4em;
  font-weight: normal;
  text-align: center;
  text-transform: uppercase;
  color: #fff;
}

.login-container {
  background: #ebebeb;
  padding: 12px;
}

/* Every row inside .login-container is defined with p tags */
.login p {
  padding: 12px;
}

.login input {
  box-sizing: border-box;
  display: block;
  width: 100%;
  border-width: 1px;
  border-style: solid;
  padding: 16px;
  outline: 0;
  font-family: inherit;
  font-size: 0.95em;
}

.login input[type="email"],
.login input[type="password"] {
  background: #fff;
  border-color: #bbb;
  color: #555;
}

/* Text fields' focus effect */
.login input[type="email"]:focus,
.login input[type="password"]:focus {
  border-color: #888;
}

.login input[type="submit"] {
  background: #28d;
  border-color: transparent;
  color: #fff;
  cursor: pointer;
}

.login input[type="submit"]:hover {
  background: #17c;
}

/* Buttons' focus effect */
.login input[type="submit"]:focus {
  border-color: #05a;
}
```

3. Abrimos nuestro app.module.ts y eliminamos las importaciones de nuestros componentes
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';


@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

4. Editamos nuestro componente login.component.ts

```typescript
import { Component, OnInit } from '@angular/core';
// Importamos las rutas
import { Router } from '@angular/router';
// Importamos nuestro servicio
import { AuthService } from '../../services/auth.service';
// Importamos nuestro modelo
import { UserI } from '../../models/user';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.css']
})
export class LoginComponent implements OnInit {
// Intyectamos nuestro servicio
  constructor(private authService: AuthService, private router: Router) { }

  ngOnInit() {
  }
// Preparamos el metodo de login
  onLogin(form): void {
    this.authService.login(form.value).subscribe(res => {
        // Si la respuesta es satisfactoria navegamos hacia el componente auth
      this.router.navigateByUrl('/auth');
    });
  }

}
```

## Trabajando con register

1. Editamos register.component.html

```html
<div class="container">
  <div class="login">
    <div class="login-triangle"></div>

    <h2 class="login-header">Create an account</h2>

    <form #frmRegister="ngForm" class="login-container" (ngSubmit)="onRegister(frmRegister)">
      <p>
        <input type="text" name="name" placeholder="name" ngModel required>
      </p>
      <p>
        <input type="email" name="email" placeholder="Email" ngModel required>
      </p>
      <p>
        <input type="password" name="password" placeholder="Password" ngModel required>
      </p>
      <p>
        <input type="submit" value="Register">
      </p>
    </form>
  </div>
</div>
```

2. Editamos register.component.ts

```typescript
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';
import { AuthService } from '../../services/auth.service';
import { UserI } from '../../models/user';

@Component({
  selector: 'app-register',
  templateUrl: './register.component.html',
  styleUrls: ['./register.component.css']
})
export class RegisterComponent implements OnInit {

  constructor(private authService: AuthService, private router: Router) { }

  ngOnInit() {
  }

  onRegister(form): void {
    this.authService.register(form.value).subscribe(res => {
      this.router.navigateByUrl('/auth');
    });
  }

}
```