---
layout: "../../layouts/BlogPostLayout.astro"
title: Fullstack Project using Next.js, Graphql and Nextauth
date: 2023-07-26
author: Teki
image: {
  src: "/images/post-7/sysImg.png",
  alt: "cover image",
}
description: A clone of Dribble including auth, graphql, image storage
draft: false
category: Coding
---

# Fullstack Project using Next.js, Graphql and Nextauth

> #### Project Resource
>
> **Build and Deploy a Full Stack Next.js 13 Application | React, Next JS 13, TypeScript, Tailwind CSS**\
> **By** ***JavaScript Mastery***\
> **Youtube Video Link:<https://www.youtube.com/watch?v=986hztrfaSQ>**\
> Thanks for amazing tutorial :)

> #### Technical Stack
>
> - Frontend: React, tailwindcss
> - Authentication: NextAuth
> - Data storage: grafbase
> - Image storage: cloudinary

> #### Main Module
>
> - Frontend UI(components)
> - Authentication(api/auth, lib/session.ts)
> - Data storage(grafbase,graphql,lib/action.ts,api/cloudinary)

![system architecture](/public/images/post-7/sysImg.png "system architecture")

### Frontend UI

> #### Tech Stack: NextAuth, GCP, Nextjs API

![project folder](/public/images/post-7/frontend_structure.png "project folder")

- app: contains api and frontend route
  - app/api: next api, can be called while app is running, in this project used as auth and image upload.
  - app/route/page.tsx: nextjs routing system, each page is a route, could be accessed by /route
- components: reusable components, such as button, navigation bar, etc.
- public: static resources, such as img, svg icon, etc. can be accessed by / from anywhere in the project
- node modules: npm package storage
- .next: store build files and cache
- .env: environment variables
- common.types.ts: export types for typescript check

### Authentication

> #### Tech Stack: NextAuth, GCP

![Auth Process](/public/images/post-7/auth_process.png "Auth Process")

#### /lib/seesion.ts

*init authOptions and getServerSession*

```javascript
import { getServerSession } from "next-auth";
import { NextAuthOptions,User } from "next-auth";
import { AdapterUser } from "next-auth/adapters";
import GoogleProvider from 'next-auth/providers/google'
import jsonwebtoken from 'jsonwebtoken'
import { JWT } from "next-auth/jwt";
import { SessionInterface, UserProfile } from "@/common.types";
import { createUser, getUser } from "./actions";

export const authOptions: NextAuthOptions = {
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret:process.env.GOOGLE_CLIENT_SECRET!
    })
  ],
  jwt:{
    encode:({secret, token})=>{
      const encodedToken = jsonwebtoken.sign({
        ...token,
        iss:'grafbase',
        exp: Math.floor(Date.now()/1000) + 60 * 60
      }, secret)
      return encodedToken
    },
    decode: async ({secret, token})=>{
      const decodedToken = jsonwebtoken.verify(token!, secret) as JWT
      return decodedToken
    }
  },
  theme:{
    colorScheme:'light',
    logo:'./logo.png'
  },
  callbacks:{
    async session({session}) {
      const email = session?.user?.email as string

      try {
        const data = await getUser(email) as {user?: UserProfile}
        const newSession = {
          ...session,
          user: {
            ...session.user,
            ...data?.user
          }
        }
        return newSession
      } catch(error) {
        console.log('Error retrieving user data',error)
        return session
      }

    },
    async signIn({user}: {user: AdapterUser | User}){
      try{
        //get the user if they exist
        const userExists = await getUser(user?.email as string) as {user?: UserProfile}
        // if they do not exist, create them
        if(!userExists.user) {
          await createUser(
            user.name as string, 
            user.email as string, 
            user.image as string
            )
        }
        return true
      } catch (error: any) {
        console.log(error)
        return false
      }
    }
  }
}

export async function getCurrentUser() {
  const session = await getServerSession(authOptions) as SessionInterface

  return session
}
```

#### /app/api/auth/[...nextauth]/route.ts

*use authOptions, generate auth handler*

```javascript
import NextAuth from "next-auth/next";
import { authOptions } from "@/lib/session";

const handler = NextAuth(authOptions)

export {handler as GET, handler as POST}
```

#### /app/api/auth/token/route.ts

*return token*

```javascript
import { getToken } from "next-auth/jwt";
import { NextRequest, NextResponse } from "next/server";

const secret = process.env.NEXTAUTH_SECRET

export async function GET(req: NextRequest) {
  const token = await getToken({req, secret, raw: true})

  return NextResponse.json({token},{status:200})
}

```

#### /app/page.tsx

*signIn page*

```javascript
"use client";
import { useState,useEffect } from "react";
import {getProviders, signIn} from 'next-auth/react'
import Button from "./Button";
type Provider = {
  id: string;
  name: string;
  type: string;
  signinUrl: string;
  callbackUrl: string;
  signinUrlParams?: Record<string, string> | undefined;
};

type Providers = Record<string, Provider>;
const AuthProviders = () => {
  const [providers, setProviders] = useState<Providers | null>(null)
  useEffect(()=>{
    const fetchProviders = async ()=>{
      const res = await getProviders();
      setProviders(res);
      console.log(res)
    }
    fetchProviders()
  },[])
  if(providers){
    return (
      <div>
        {Object.values(providers).map(
          (provider: Provider, i)=> (
            <Button key={i} title="Sign In" handleClick={()=>signIn(provider?.id)} />
          )
        )}
      </div>
    )
  }
}

export default AuthProviders
```

### Image Storage

> #### Tech Stack: Cloudinary, Nextjs API

![Image Storage](/public/images/post-7/cloudinary.png "Image Storage")

#### /app/api/upload/route.ts

```javascript
import { NextResponse } from "next/server"
import {v2 as cloudinary} from 'cloudinary'

cloudinary.config({ 
  cloud_name: process.env.CLOUDINARY_NAME, 
  api_key: process.env.CLOUDINARY_KEY, 
  api_secret: process.env.CLOUDINARY_SECRET 
});


export async function POST(request:Request) {
    const {path} = await request.json()
    if(!path){
      return NextResponse.json(
        {message: "Image path is required"},
        {status: 400}
      )
    }

    try {
      const options = {
        use_filename: true,
        unique_filename: false,
        overwrite: true,
        transformation: [{width: 1000, height: 752, crop:'scale'}]
      }
      const result = await cloudinary.uploader.upload(path,options)
      return NextResponse.json(result,{status:200})
    } catch (error) {
        return NextResponse.json(
        {message: error},
        {status: 500}
      )
    }
}
```

#### /components/ProjectForm.tsx(only image related part)

```javascript
"use client"
import { ProjectInterface, SessionInterface } from "@/common.types"
import { ChangeEvent, useState } from "react"
import Image from "next/image"
import FormField from "./FormField"
import { categoryFilters } from "@/constants"
import CustomMenu from "./CustomMenu"
import Button from "./Button"
import { createNewProject, fetchToken, updateProject } from "@/lib/actions"
import { useRouter } from "next/navigation"
type Props = {
  type: string,
  session: SessionInterface,
  project?: ProjectInterface
}
const ProjectForm = ( {type, session, project}: Props ) => {
  const router = useRouter()
  const handleFormSubmit = async  (e: React.FormEvent) => {
    e.preventDefault()
    setIsSubmitting(true)
    const {token} = await fetchToken()
    try {
      if(type==='create'){
        // create new project
        await createNewProject(form, session?.user?.id, token)
        router.push('/')
      }
      if(type === 'edit'){
        await updateProject(form, project?.id as string, token)
        router.push('/')
      }
    } catch (error) {
      console.log(error)
    } finally {
      setIsSubmitting(false)
    }
  }


  const handleChangeImage = (e: ChangeEvent<HTMLInputElement>) => {
    e.preventDefault()
    const file = e.target.files?.[0]
    if(!file) return
    if(!file.type.includes('image')) {
      return alert("Please upload an image file")
    }
    const reader = new FileReader()
    reader.readAsDataURL(file)
    reader.onload = ()=>{
      const result = reader.result as string
      handleStateChange('image',result)
    }
  }

  return (
    <form onSubmit={handleFormSubmit} className="flexStart form">
        <label htmlFor="poster" className="flexCenter form_image-label">
          {!form.image && 'Choose a poster for your project'}
        </label>
        <input id="image" type="file" accept="image/*" required={type==='create'} className="form_image-input" onChange={handleChangeImage}/>
        {form.image && (
          <Image src={form?.image} className="sm:p-10 object-contain z-20" alt="Project poster" fill/>
        )}
      </div>
      <div className="flexStart w-full">
        <Button 
          title={isSubmitting ? `${type==='create'?'Creating':'Editing'}`
          :`${type==='create'?'Create':'Edit'}`} 
          type="submit" leftIcon={isSubmitting?"":"/plus.svg"} submitting={isSubmitting}
        />
      </div>
    </form>
  )
}

export default ProjectForm
```

#### /lib/action.ts/uploadImage

```javascript
export const uploadImage = async (imagePath: string) => {
  try {
    const response = await fetch(`${serverUrl}/api/upload`,{
      method: 'POST',
      body: JSON.stringify({path: imagePath})
    })
    return response.json()
  } catch (error) {
    throw error
  }
}
```

### Data Storage (Graphql)

> #### Tech Stack: Grafbase, graphql-request

![Data Storage](/public/images/post-7/grafbase.png "Data Storage")

- #### Graphql Client

make graphql query, connect grafbase

#### /lib/action.ts

```javascript
import { ProjectForm } from "@/common.types";
import { allProjectsQuery, createProjectMutation, createUserMutation, deleteProjectMutation, getProjectByIdQuery, getProjectsOfUserQuery, getUserQuery, projectsQuery, updateProjectMutation } from "@/graphql";
import { GraphQLClient } from "graphql-request";

const isProduction = process.env.NODE_ENV === 'production';
const apiUrl = isProduction ? process.env.NEXT_PUBLIC_GRAFBASE_API_URL || '' : 'http://127.0.0.1:4000/graphql'
const apiKey = isProduction ? process.env.NEXT_PUBLIC_GRAFBASE_API_KEY || '' : 'letmein'
const serverUrl = isProduction ? process.env.NEXT_PUBLIC_SERVER_URL : 'http://localhost:3000'
const client = new GraphQLClient(apiUrl)
const makeGraphQLRequest = async (query: string, variables={}) => {
  try{
    //client request....
    return await client.request(query, variables)
  } catch (error) {
    throw error
  }
}

export const getUser = (email: string) => {
  client.setHeader('x-api-key',apiKey)
  return makeGraphQLRequest(getUserQuery,{email})
}

export const createUser = (name: string, email: string, avatarUrl: string) => {
  client.setHeader('x-api-key',apiKey)
  const variables = {
    input: {
      name, email, avatarUrl
    }
  }
  return makeGraphQLRequest(createUserMutation, variables)
}

export const fetchToken = async ()=> {
  try {
    const response = await fetch(`${serverUrl}/api/auth/token`)
    return response.json()
  } catch(error) {
    throw error
  }
}

export const uploadImage = async (imagePath: string) => {
  try {
    const response = await fetch(`${serverUrl}/api/upload`,{
      method: 'POST',
      body: JSON.stringify({path: imagePath})
    })
    return response.json()
  } catch (error) {
    throw error
  }
}

export const createNewProject = async (form: ProjectForm, creatorId: string, token: string) => {
  const imageUrl = await uploadImage(form.image)
  if(imageUrl.url) {
    client.setHeader("Authorization",`Bearer ${token}`)
    const variables = {
      input: {
        ...form,
        image: imageUrl.url,
        createdBy: {
          link: creatorId
        }
      }
    }
    return makeGraphQLRequest(createProjectMutation, variables)
  }
}


export const fetchAllProjects = (category?: string | null, endcursor?: string | null) => {
  client.setHeader("x-api-key", apiKey);
  if(!category){
    return makeGraphQLRequest(allProjectsQuery)
  } else{
    return makeGraphQLRequest(projectsQuery,{category,endcursor});
  }
};


export const getProjectDetails = (id:string) => {
  client.setHeader('x-api-key',apiKey)
  return makeGraphQLRequest(getProjectByIdQuery,{id})
}


export const getUserProjects = (id:string, last?: number) => {
  client.setHeader('x-api-key',apiKey)
  return makeGraphQLRequest(getProjectsOfUserQuery,{id, last})
}

export const deleteProject = (id:string, token: string) => {
  client.setHeader("Authorization",`Bearer ${token}`)
  return makeGraphQLRequest(deleteProjectMutation,{id})
}

export const updateProject = async (form: ProjectForm, projectId:string, token: string) => {
  function isBase64DataURL(value: string) {
    const base64Regex = /^data:image\/[a-z]+;base64,/;
    return base64Regex.test(value);
  }
  let updatedForm = {...form}
  const isUploadingNewImage = isBase64DataURL(form.image)
  if(isUploadingNewImage) {
    const imageUrl = await uploadImage(form.image)
    if(imageUrl.url) {
      updatedForm = {
        ...form,
        image: imageUrl.url
      }
    }
  }
  const variables = {
    id: projectId,
    input: updatedForm
  }
  client.setHeader("Authorization",`Bearer ${token}`)
  return makeGraphQLRequest(updateProjectMutation,variables)
}

```

- #### graphql query

store graphql query

#### /graphql/index.ts

```javascript
export const createProjectMutation = `
 mutation CreateProject($input: ProjectCreateInput!) {
  projectCreate(input: $input) {
   project {
    id
    title
    description
    createdBy {
     email
     name
    }
   }
  }
 }
`;

export const updateProjectMutation = `
 mutation UpdateProject($id: ID!, $input: ProjectUpdateInput!) {
  projectUpdate(by: { id: $id }, input: $input) {
   project {
    id
    title
    description
    createdBy {
     email
     name
    }
   }
  }
 }
`;

export const deleteProjectMutation = `
  mutation DeleteProject($id: ID!) {
    projectDelete(by: { id: $id }) {
      deletedId
    }
  }
`;
      
export const createUserMutation = `
 mutation CreateUser($input: UserCreateInput!) {
  userCreate(input: $input) {
   user {
    name
    email
    avatarUrl
    description
    githubUrl
    linkedInUrl
    id
   }
  }
 }
`;
export const allProjectsQuery = `
  query getProjects {
    projectSearch(first: 1) {
      pageInfo {
        hasNextPage
        hasPreviousPage
        startCursor
        endCursor
      }
      edges {
        node {
          title
          githubUrl
          description
          liveSiteUrl
          id
          image
          category
          createdBy {
            id
            email
            name
            avatarUrl
          }
        }
      }
    }
  }
`;
export const projectsQuery = `
  query getProjects($category: String, $endcursor: String) {
    projectSearch(first: 8, after: $endcursor, filter: {category: {eq: $category}}) {
      pageInfo {
        hasNextPage
        hasPreviousPage
        startCursor
        endCursor
      }
      edges {
        node {
          title
          githubUrl
          description
          liveSiteUrl
          id
          image
          category
          createdBy {
            id
            email
            name
            avatarUrl
          }
        }
      }
    }
  }
`;


export const getProjectByIdQuery = `
  query GetProjectById($id: ID!) {
    project(by: { id: $id }) {
      id
      title
      description
      image
      liveSiteUrl
      githubUrl
      category
      createdBy {
        id
        name
        email
        avatarUrl
      }
    }
  }
`;

export const getUserQuery = `
  query GetUser($email: String!) {
    user(by: { email: $email }) {
      id
      name
      email
      avatarUrl
      description
      githubUrl
      linkedInUrl
    }
  }
`;
      
export const getProjectsOfUserQuery = `
  query getUserProjects($id: ID!, $last: Int = 4) {
    user(by: { id: $id }) {
      id
      name
      email
      description
      avatarUrl
      githubUrl
      linkedInUrl
      projects(last: $last) {
        edges {
          node {
            id
            title
            image
          }
        }
      }
    }
  }
`;
```

- #### grafbase

init grafbase and data model

#### /grafbase/grafbase.config.ts

```javascript
import { g, auth, config } from '@grafbase/sdk'


// Welcome to Grafbase!
// Define your data models, integrate auth, permission rules, custom resolvers, search, and more with Grafbase.
// Integrate Auth
// https://grafbase.com/docs/auth
//
// const authProvider = auth.OpenIDConnect({
//   issuer: process.env.ISSUER_URL ?? ''
// })
//
// Define Data Models
// https://grafbase.com/docs/database

// @ts-ignore
const User = g.model('User',{
  name: g.string().length({ min: 2, max: 20}),
  email: g.string().unique(),
  avatarUrl: g.url(),
  description: g.string().optional(),
  githubUrl: g.url().optional(),
  linkedInUrl: g.url().optional(),
  projects: g.relation(()=>Project).list().optional(),
}).auth((rules)=>{
  rules.public().read()
})

// @ts-ignore
const Project = g.model('Project',{
  title: g.string().length({min:3}),
  description: g.string(),
  image: g.url(),
  liveSiteUrl:g.url(),
  githubUrl: g.url(),
  category: g.string().search(),
  createdBy: g.relation(()=>User)
}).auth((rules)=>{
  rules.public().read(),
  rules.private().create().delete().update();
})

const jwt = auth.JWT({
  issuer: 'grafbase',
  secret: g.env('NEXTAUTH_SECRET')
})


export default config({
  schema: g,
  // Integrate Auth
  // https://grafbase.com/docs/auth
  auth: {
    providers: [jwt],
    rules: (rules) => {
      rules.private()
    }
  }
})

```

#### /grafbase/.env

```
# KEY=VALUE
NEXTAUTH_SECRET = abc
```

#### An example for data fetch at frontend

#### /app/page.tsx

```javascript
const Home = async ({searchParams: {category, endCursor}}: Props) => {
  const data = await fetchAllProjects(category,endCursor) as ProjectSearch
  const projectsToDisplay = data?.projectSearch?.edges || []
  return (
    <div>
    {projectsToDisplay}
    </div>
  )
}

// submit and store a new project(need token)
const handleFormSubmit = async  (e: React.FormEvent) => {
    e.preventDefault()
    setIsSubmitting(true)
    const {token} = await fetchToken()
    try {
      if(type==='create'){
        // create new project
        await createNewProject(form, session?.user?.id, token)
        router.push('/')
      }
      if(type === 'edit'){
        await updateProject(form, project?.id as string, token)
        router.push('/')
      }
    } catch (error) {
      console.log(error)
    } finally {
      setIsSubmitting(false)
    }
  }
```
