---
title: 'Authentification'
description: 'Authentification'
pubDate: 'Jan 26 2025'
heroImage: '/auth-placeholder.jpg'
---

### Authentification

After I finished the logic of the app, I decided to add the authentification to the app. I wanted to add the authentification to the app, so the user can have his own account and see the statistics of his flights. For this purpose I decided to use the Supabase Authentification.

I decided to use the Supabase Authentification because it is a very simple and easy to use authentification service. It provides the authentification with email and password, social authentification, and also the JWT authentification. Also, it provides the user management, so I can easily manage the users and their data.

Firebase was also a good option, but I decided to use Supabase because it is more open-source and I can easily manage the data in the Postgres database.

### Setting up the Supabase Authentification

So, to start using the Supabase Authentification, I needed to create the account on the Supabase website and create the new project. After that, I needed to get the API key and the URL of the project. I added the API key and the URL to the environment variables.

After that, I installed the Supabase library to the project. I created the new service called `auth.service.ts` and added the authentification logic to it.

All the logic of auth and functions I took from the Supabase tutorial specifically for Ionic framework. I just needed to adjust the code a bit to make it work with my app.

The tutorial is available [here](https://supabase.com/docs/guides/getting-started/tutorials/with-ionic-angular).

```typescript
//supabase.service.ts

import { Injectable } from '@angular/core';
import {
  createClient,
  SupabaseClient,
  AuthChangeEvent,
  Session,
} from '@supabase/supabase-js';
import { environment } from '../../environments/environment';
import { LoadingController, ToastController } from '@ionic/angular';

export interface Profile {
  username: string;
  first_name: string;
  avatar_url: string;
}

@Injectable({
  providedIn: 'root',
})
export class SupabaseService {
  private supabase: SupabaseClient;

  constructor(
    private loadingCtrl: LoadingController,
    private toastCtrl: ToastController
  ) {
    this.supabase = createClient(environment.supabaseUrl, environment.supabaseKey, {
    auth: {
      persistSession: true,
      detectSessionInUrl: true,
      autoRefreshToken: false,
    },
  });
  }

  get user() {
    return this.supabase.auth.getUser().then(({ data }) => data?.user);
  }

  signIn(email: string, password: string) {
    return this.supabase.auth.signInWithPassword({ email, password });
  }

  signOut() {
    return this.supabase.auth.signOut();
  }

  async signUp(email: string, password: string, profile: Partial<Profile>) {
    const { data, error } = await this.supabase.auth.signUp({ email, password });

    if (error) {
      throw error;
    }

    if (!data.user) {
      throw new Error('User was not created successfully');
    }

    const { error: profileError } = await this.supabase
      .from('profiles')
      .insert([
        {
          id: data.user.id,
          // email: data.user.email,
          first_name: profile.first_name || '',
          username: profile.username || email.split('@')[0],
          avatar_url: profile.avatar_url || 'https://ionicframework.com/docs/img/demos/avatar.svg',
        },
      ]);

    if (profileError) {
      throw new Error(`Profile creation failed: ${profileError.message}`);
    }

    const { error: signInError } = await this.supabase.auth.signInWithPassword({
      email,
      password,
    });

    if (signInError) {
      throw new Error(`Signup successful, but login failed: ${signInError.message}`);
    }

    return data.user;
  }



  async uploadAvatar(file: File): Promise<string> {
    const user = await this.user;
    if (!user) throw new Error('No user is logged in');

    const fileName = `${user.id}/${file.name}`;
    const { data: uploadData, error: uploadError } = await this.supabase.storage
      .from('avatars')
      .upload(fileName, file);

    if (uploadError) {
      throw new Error(`Avatar upload failed: ${uploadError.message}`);
    }

    const { data: publicUrlData } = this.supabase.storage
      .from('avatars')
      .getPublicUrl(fileName);
    if (!publicUrlData) {
      throw new Error(`Failed to retrieve public URL.`);
    }

    return publicUrlData.publicUrl;
  }


  async updateProfile(profile: Profile) {
    const user = await this.user;
    const update = {
      ...profile,
      id: user?.id,
      updated_at: new Date(),
    };
    return this.supabase.from('profiles').upsert(update);
  }

  async getProfile() {
    const user = await this.user; 
    if (!user) {
      throw new Error('No user is logged in');
    }

    const { data, error } = await this.supabase
      .from('profiles') 
      .select('id, username, first_name, avatar_url')
      .eq('id', user.id)
      .single();

    if (error) {
      throw error;
    }

    return {
      id: data.id,
      username: data.username,
      first_name: data.first_name,
      avatar_url: data.avatar_url,
    };
  }

  async createLoader() {
    const loader = await this.loadingCtrl.create();
    return loader;
  }

  async createNotice(message: string) {
    const toast = await this.toastCtrl.create({ message, duration: 3000 });
    toast.present();
  }
}


```

### Profile page
To show the user profile, I used the new page called `tab3.page.ts`. In this page I added the logic to show the user profile.

```typescript
//tab3.page.ts

import { Component, OnInit } from '@angular/core';
import { ExploreContainerComponent } from '../explore-container/explore-container.component';
import { CommonModule } from '@angular/common';
import { SupabaseService, Profile } from '../services/supabase.service';
import { FormsModule } from '@angular/forms';

import {
  IonContent,
  IonHeader,
  IonTitle,
  IonToolbar,
  IonButtons,
  IonBackButton,
  IonCard,
  IonRow,
  IonCol,
  IonGrid,
  IonCardHeader,
  IonCardTitle,
  IonCardContent,
  IonIcon,
  IonItem,
  IonLabel,
  IonInput,
  IonButton,
} from '@ionic/angular/standalone';

@Component({
  selector: 'app-tab3',
  templateUrl: 'tab3.page.html',
  styleUrls: ['tab3.page.scss'],
  standalone: true,
  imports: [
      IonContent,
      IonHeader,
      IonTitle,
      IonToolbar,
      FormsModule,
      IonButtons,
      IonGrid,
      IonCol,
      IonRow,
      IonBackButton,
      IonCard,
      IonIcon,
      IonCardHeader,
      IonCardTitle,
      IonCardContent,
      IonItem,
      IonLabel,
      IonInput,
      IonButton,
      CommonModule,
    ],
})
export class Tab3Page implements OnInit {
  profile: Profile = {
    username: '',
    first_name: '',
    avatar_url: '',
  };

  email = ''; 


  constructor(private supabase: SupabaseService) {}

  ngOnInit() {
    this.loadProfile();
  }

  
  async loadProfile() {
    try {
      const profile = await this.supabase.getProfile(); 
      if (profile) {
        this.profile = { ...this.profile, ...profile }; 
        console.log('Profile:', this.profile);
      }
    } catch (error: any) {
      console.error('Failed to load profile:', error.message);
    }
  }

  async signOut() {
    await this.supabase.signOut();
  }
}
```

### Login and Sign up form
To handle the login and sign up form, I used the new page called `login.page.ts` and `signup.page.ts`. In this page I added the logic to handle the login and sign up form.

```typescript
//login.page.ts

import { Component } from '@angular/core';
import { NavController } from '@ionic/angular';
import { SupabaseService } from '../../services/supabase.service';
import { CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { 
  IonContent,
  IonHeader,
  IonTitle,
  IonToolbar,
  IonButtons,
  IonBackButton,
  IonCard,
  IonRow,
  IonCol,
  IonGrid,
  IonCardHeader,
  IonCardTitle,
  IonCardContent,
  IonIcon,
  IonItem,
  IonLabel,
  IonInput,
  IonButton,
} from '@ionic/angular/standalone';

@Component({
  selector: 'app-login',
  standalone: true,
  templateUrl: './login.page.html',
  styleUrls: ['./login.page.scss'],
  imports: [
    IonContent,
    IonHeader,
    IonTitle,
    IonToolbar,
    IonButtons,
    IonGrid,
    IonCol,
    IonRow,
    IonBackButton,
    FormsModule,
    IonCard,
    IonIcon,
    IonCardHeader,
    IonCardTitle,
    IonCardContent,
    IonItem,
    IonLabel,
    IonInput,
    IonButton,

  ],
  // schemas: [
  //   CUSTOM_ELEMENTS_SCHEMA,
  // ]
})
export class LoginPage {
  email = '';
  password = '';

  constructor(
    private navCtrl: NavController,
    private supabase: SupabaseService
  ) {}

  async login() {
    console.log('Login button clicked');
    const loader = await this.supabase.createLoader();
    await loader.present();

    try {
      const { data, error } = await this.supabase.signIn(this.email, this.password);
      if (error) {
        throw new Error(error.message);
      }

      console.log('Login successful:', data.session);
      this.supabase.createNotice('Login successful!');
      this.navCtrl.navigateRoot('/tabs/tab1');
    } catch (err: any) {
      console.error('Login failed:', err.message);
      this.supabase.createNotice(err.message);
    } finally {
      loader.dismiss();
    }
  }

  goToSignUp() {
    this.navCtrl.navigateForward('/auth/signup');
  }
}

```

```typescript
//signup.page.ts

import { Component } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { NavController } from '@ionic/angular';
import { SupabaseService } from '../../services/supabase.service';
import {
  IonContent,
  IonHeader,
  IonTitle,
  IonToolbar,
  IonButtons,
  IonBackButton,
  IonCard,
  IonRow,
  IonCol,
  IonGrid,
  IonCardHeader,
  IonCardTitle,
  IonAvatar,
  IonCardContent,
  IonIcon,
  IonItem,
  IonLabel,
  IonInput,
  IonButton,
} from '@ionic/angular/standalone';
import { FormsModule } from '@angular/forms';


@Component({
  selector: 'app-signup',
  standalone: true,
  templateUrl: './signup.page.html',
  styleUrls: ['./signup.page.scss'],
  imports: [
    IonContent,
    IonHeader,
    IonTitle,
    IonToolbar,
    IonButtons,
    IonGrid,
    IonCol,
    IonAvatar,
    IonRow,
    IonBackButton,
    IonCard,
    IonIcon,
    IonCardHeader,
    IonCardTitle,
    IonCardContent,
    IonItem,
    IonLabel,
    IonInput,
    IonButton,
    FormsModule,
  ],
})
export class SignupPage {
  email = '';
  password = '';
  firstName = '';
  username = '';
  avatarFile: File | null = null;
  previewAvatar: string | ArrayBuffer | null = null;

  constructor(private navCtrl: NavController, private supabase: SupabaseService) {}

  onFileSelected(event: any) {
    const file = event.target.files[0];
    this.avatarFile = file;

    const reader = new FileReader();
    reader.onload = (e) => {
      this.previewAvatar = e.target?.result ?? null; 
    };
    reader.readAsDataURL(file);
  }


  async signUp() {
    try {
      let avatarUrl = '';
      if (this.avatarFile) {
        avatarUrl = await this.supabase.uploadAvatar(this.avatarFile);
      }

      const user = await this.supabase.signUp(this.email, this.password, {
        first_name: this.firstName,
        username: this.username,
        avatar_url: avatarUrl,
      });

      if (user) {
        this.supabase.createNotice('Signup successful! You are logged in.');
        this.navCtrl.navigateRoot('/tabs/tab1');
      }
    } catch (error: any) {
      console.error('Sign-up failed:', error.message);
      this.supabase.createNotice(error.message);
    }
  }


  goToLogin() {
    this.navCtrl.navigateForward('/auth/login');
  }
}
```