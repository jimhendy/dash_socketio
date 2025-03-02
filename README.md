# dash socketio

Client Socket.IO for Dash

Forked from https://github.com/RenaudLN/dash_socketio who did all the hard work.

I wanted to be able to use the socketio client in a Dash app without having to use the Flask server. This is a simple way to do that. We can now simply return socket events from the Dash app and they will be sent to the client.

## Demo app

Run the client and server separately:

### Server
```bash
uv run -p 3.12 --with flask,flask_socketio,loguru usage_server.py
```

### Client
```bash
uv run -p 3.12 --with dash_bootstrap_components,"dash_mantine_components==0.12",loguru usage_client.py
```

## Usage

The server does not use this component, it is only for the client.
The server uses a standard flask_socketio instance.

The client is a Dash component that can be used to send and receive events from a socket.io server.
The `DashSocketIO` component is included in the dash layout and can be used to send and receive events.
It has no visual representation.

Client Callbacks can output to the `DashSocketIO` component to send events to the server.
The `DashSocketIO` component can also be used as an input to a callback to receive events from the server.

```python
import dash
from dash import html
from dash.dependencies import Input, Output
from dash_socketio import DashSocketIO

app = dash.Dash(__name__)

app.layout = html.Div([
    DashSocketIO(id='socketio'),
    html.Button('Send', id='send'),
    html.Div(id='output')
])

@app.callback(Output('output', 'children'), Input('socketio', 'event'))
def handle_event(event):
    return event

@app.callback(Output('socketio', 'event'), Input('send', 'n_clicks'))
def send_event(n_clicks):
    if n_clicks:
        return {"event": "my_event", "data": "my_data", "id": uuid.uuid4().hex}

if __name__ == '__main__':
   app.run_server(debug=True)
```

The `"event"` returned  is used by the server to distribute the event to the correct handler.
The `"data"` returned is the data that will be sent to the server.
The `"id"` returned is a purely infrastructe annoyance but required as otherwise the server may not act on consecutive, otherwise identical events.

We recommend using the utility function `websocket_requet` to send events to the server.

```python
import uuid

def websocket_request(
    event: str,
    data: Any = None,
):
    """
    Simple helper function to create a websocket request.

    This is required because we need to alter the "send" property of the
    DashSocketIO component so we had a uuid to each request.
    Otherwise, if we requested two identical events in a row, the second
    request would be ignored.

    :param event: The event to send
    :param data: The data to send with the event. Defaults to None. Usually a dict.
    :return: A dictionary with the event, data, and a uuid.
    """
    return {
        "event": event,
        "data": data,
        "id": str(uuid.uuid4()),
    }
```