##### 断线重连，心跳监测
1、心跳监测,每10s心跳检测保活
```
function checkSocketConn() {
      //每10s检测一下是否断连
      checker = setInterval(() => {
        conn?.instance?.send('ping')
        dispatch(updateTimeAction(dayjs().format('MM-DD HH:mm:ss')))
      }, 10 * 1000)
    }
```
2、每收到socket消息后发送确认回复

```
   conn.instance.addEventListener('message', (e: { data: string }) => {
          const msg = JSONParse<msgProps>(e.data, {})
          log('websocketModel__message', e.data)
          if (msg !== null) {
            eventDispatches.forEach(({ test, name }: testProps) => {
              if (test(msg)) {
                dispatch({ type: name, payload: msg })
                // 上报消息类型,收到消息处理后进行上报
                if (!conn) return
                conn.instance.send([msg.MsgId, msg.GroupId, msg.From, msg.To, Date.now(), name].join('|'))
              }
            })
          }
        })
```

3、断线重连
使用reconnecting-websocket库，会自动断开重连
```
 this.instance = new ReconnectingWebsocket(this.url, [], {
      debug: this.options.debug,
    })
```

4、对于连接状态做记录
记录连接状态：
```
export const WEBSOCKET_STATE = {
  CONNECTING: 'CONNECTING',
  OPEN: 'OPEN',
  REOPEN: 'REOPEN',
  ERROR: 'ERROR',
  CLOSE: 'CLOSE',
}
```

记录上次连接状态更新时间：
```
const state = {
  websocketState: null,
  websocketError: null,
  websocketClose: null,
  websocketTime: '',
  websocketReopen: false,
}

```

监听到open,message,error,close等状态时做状态更新，在页面上做出连接状态提示


##### 如何处理异步消息
redux-saga处理异步
window.sagaMiddleware.run(rootSaga)
yield spawn(chatRootSaga)

```
function* newMessageSaga(action) {
  try {
    log('newTextMessage', action)
    const msg = inputMessage(action.payload)
    const { groupID: userID } = msg

    if (msg?.fromType === 'user') {
      yield fork(playSound, 'new_msg')

      yield put(checkMessageForUsersAction(msg))
    }

    yield put(pushMessage({ target: { userID }, payload: [msg], meta: { newMessage: true } }))
  }
}
```
##### 如何保证消息顺序
用户消息列队
```
function* newMessageSaga(action) {
  try {
    log('newTextMessage', action)
    const msg = inputMessage(action.payload)
    const { groupID: userID } = msg

    if (msg?.fromType === 'user') {
      yield fork(playSound, 'new_msg')

      yield put(checkMessageForUsersAction(msg))
    }

    yield put(pushMessage({ target: { userID }, payload: [msg], meta: { newMessage: true } }))
  }
}
```

key值是messageID


##### 发送消息
```
function* sendMsgToUser(action) {
  try {
    log('sendMsgToUser listener', action)
    const { payload } = action
    const user = yield select((state) => state.user)
    payload.Content = payload.Content || ''
    if (typeof payload.Content === 'string') {
      payload.Content = payload.Content.trim()
    }

    if (!payload.Content && !payload.KfPicUrl && !payload.KfFileUrl && !payload.MediaId) {
      Message.error('发送内容不能为空')
      return
    }

    const userID = payload.userid
    let message = {
      To: payload.userid,
      FromType: 'agent',
      From: `${user.info.corpID}_${user.info.userID}`,
      Content: payload.Content,
      MsgType: payload.MsgType,
      KfPicUrl: payload.KfPicUrl,
      KfFileUrl: payload.KfFileUrl || '',
      MediaId: payload.MediaId,
    }

    log('sendMsgToUser begin', userID, message, action)

    const { msgID, msgTime } = yield call(
      api.sendMsgToUser,
      { corpid: user.info.corpID.toString(), token: user.info.token },
      message
    )

    message.MsgId = msgID
    message.groupID = payload.userid
    message.msgTime = msgTime || parseInt(new Date().getTime() / 1000)
    message = inputMessage(message)
    yield put(sendMsgToUserSuccessAction(message))
    yield put(pushMessage({ target: { userID }, payload: [message] }))
    log('sendMsgToUser success', userID, message)
  } catch (e) {
    let msg = '接口异常'
    if (e.data && e.data.ErrMsg) {
      msg = e.data.ErrMsg
    }
    Message.error(msg)
    error('sendMsgToUser error', action, e?.data)
  }
}
```