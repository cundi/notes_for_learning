#  理解JWT（JSON Web Token）认证及python实践
https://segmentfault.com/a/1190000010312468

## sanic sample

JWT structure
Token itself consists of three parts: header, payload and signature:

header contains encoded type of the token and algorithm
payload contains encoded data and additional metadata
signature is encoded header + encoded data + secret phrase, encoded with algorithm

JWT vs Cookie-based Sessions
JWT brings some benefits compared to cookie-based authentication:

Easy cross-domain requests.
Server-side scalability. Since JWT token carries all information inside, token and secret phrase is all that is required to authenticate a user.
Weak coupling. As JWT is stateless and supported by standard, it is possible to use it with different services, possibly written with several languages.
Easily usable in mobile development. No cookie emulation is required for mobile clients.
No CSRF. No automatic cookie is passed by browser - no vulnerability.
Standard-based. JWT is well supported by all major languages.

Next, create a handler to allow the client to login, i.e. acquire authentication token.

```
from datetime import datetime, timedelta
import jwt

JWT_SECRET = 'secret'
JWT_ALGORITHM = 'HS256'
JWT_EXP_DELTA_SECONDS = 20

async def login(request):
    post_data = await request.post()

    try:
        user = User.objects.get(email=post_data['email'])
        user.match_password(post_data['password'])
    except (User.DoesNotExist, User.PasswordDoesNotMatch):
        return json_response({'message': 'Wrong credentials'}, status=400)

    payload = {
        'user_id': user.id,
        'exp': datetime.utcnow() + timedelta(seconds=JWT_EXP_DELTA_SECONDS)
    }
    jwt_token = jwt.encode(payload, JWT_SECRET, JWT_ALGORITHM)
    return json_response({'token': jwt_token.decode('utf-8')})

app = web.Application()
app.router.add_route('POST', '/login', login)
````


Here is a step-by-step breakdown of the process:

Get data from post query.
Fetch user by email from storage.
Check if passwords match.
If the user with the email does not exist or the password doesn't match, return response with error message.
Next, create token payload, where we store data we'd like to have when authorized clients perform certain actions. There are reserved keys, like exp, which JWT standard defines and its implementations use internally to provide additional features. In our case, we store the user ID to identify user and expiration date, after which the token becomes invalid. Description of exp and other reserved keys provided in corresponding RFC section
Finally, we encode our payload with a secret string and specified algorithm and the return response with a token in the JSON body.


async def get_user(request):
    return json_response({'user': str(request.user)})

async def auth_middleware(app, handler):
    async def middleware(request):
        request.user = None
        jwt_token = request.headers.get('authorization', None)
        if jwt_token:
            try:
                payload = jwt.decode(jwt_token, JWT_SECRET,
                                     algorithms=[JWT_ALGORITHM])
            except (jwt.DecodeError, jwt.ExpiredSignatureError):
                return json_response({'message': 'Token is invalid'}, status=400)

            request.user = User.objects.get(id=payload['user_id'])
        return await handler(request)
    return middleware

app = web.Application(middlewares=[auth_middleware])
app.router.add_route('GET', '/get-user', get_user)
Let’s go through it step-by-step:

Define the aiohttp middleware.
Get token from AUTHORIZATION header, if there is one.
Try to decode it with the same secret and encoding algorithm as it was created.
If token expired or decoding error occurs, return response with error message.
If everything OK, fetch user by with user_id in payload and assign it to request.user.
Note that middlewares=[auth_middleware] added to Application instance creation.