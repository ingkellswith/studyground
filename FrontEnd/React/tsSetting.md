React TypeScript 세팅
================
react에 typescript를 적용하고 싶다면  
처음부터 타입스크립트 프로젝트로 적용시켜 주는 방법이 있고,  
javascript를 사용하는 코드를 타입스크립트로 전환시켜주는 방법이 있다.

### 첫 번째 방법 : 처음부터 타입스크립트 프로젝트로 만들기
    npx create-react-app app-name --template typescript  

이 명령어만 입력하면 자동으로 타입스크립트 환경을 세팅해준다.  
너무 쉬우므로 js를 ts로 전환하는 두 번째 방법으로 넘어가자.

### 두 번째 방법 : js리액트 프로젝트를 ts리액트 프로젝트로 전환하기  

리액트 프로젝트를 진행하다 보면 babel과 webpack이 필요한 상황이 올 것이다.  
그 떄 ts-loader를 사용하면 쉽게 설정할 수 있으므로 ts-loader를 기반으로 세팅한
ts환경을 예시로 보여주겠다.  
아래는 현재 진행하고 있는 토이프로젝트의 초기 세팅 파일이다.  

### package.json
    {
        "name": "rochest-frontend",
        "version": "0.1.0",
        "dependencies": {
            "@testing-library/jest-dom": "^5.11.4",
            "@testing-library/react": "^11.1.0",
            "@testing-library/user-event": "^12.1.10",
            "@types/jest": "^27.0.1",
            "@types/node": "^14.6.1",
            "@types/react": "^16.9.48",
            "@types/react-dom": "^17.0.9",
            "react": "^17.0.2",
            "react-dom": "^17.0.2",
            "react-router-dom": "^5.2.0",
            "react-scripts": "4.0.3",
            "ts-loader": "^8.0.3",
            "typescript": "^3.9.7",
            "web-vitals": "^1.0.1"
        },
        "scripts": {
            "start": "react-scripts start",
            "build": "react-scripts build",
            "test": "react-scripts test",
            "eject": "react-scripts eject"
        },
        "eslintConfig": {
            "extends": [
            "react-app",
            "react-app/jest"
            ]
        },
        "browserslist": {
            "production": [
            ">0.2%",
            "not dead",
            "not op_mini all"
            ],
            "development": [
            "last 1 chrome version",
            "last 1 firefox version",
            "last 1 safari version"
            ]
        }
    }  

### tsconfig.json
    {
        "extends": "./tsconfig-base.json",
        "files": [
            "src/index.tsx"
        ],
        "compilerOptions": {
            "lib": [
            "dom",
            "dom.iterable",
            "esnext"
            ],
            "allowJs": true,
            "skipLibCheck": true,
            "allowSyntheticDefaultImports": true,
            "strict": true,
            "forceConsistentCasingInFileNames": true,
            "noFallthroughCasesInSwitch": true,
            "module": "esnext",
            "resolveJsonModule": true,
            "isolatedModules": true,
            "noEmit": true
        },
        "include": [
            "src"
        ]
    }

### tsconfig-base.json
    {
        "compilerOptions": {
        "declaration": true,
        "declarationMap": true,
        "target": "es5",
        "module": "commonjs",
        "moduleResolution": "node",
        "esModuleInterop": true,
        "composite": true,
        "jsx": "react",
        }
    }

이렇게 세팅한 후 기존 js파일들을 tsx파일로 파일명만 바꿔주고,  
추가적으로 바벨이나 웹펙이 필요하면 문서를 읽고 설정파일만 바꿔주면 될 것이다.    


ts-loader demo github : https://github.com/TypeStrong/ts-loader/tree/3f3c0d779a9df614b978764e976d934d08f1fdca/examples/project-references-example


