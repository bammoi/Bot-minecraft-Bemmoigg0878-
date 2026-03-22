const { createClient } = require('bedrock-protocol')

const SERVER = {
  host: 'pe.toromc.vn',
  port: 19132,
  username: 'Bemmoigg0878'
}

let bot

let dayTimer
let reconnectTimer
let smpCheckTimer
let hubCheckTimer

let lastShardLine = false
let inSMP = false
let hubReady = false



function debug(msg){
  console.log('[DEBUG]', msg)
}



function startBot(){

  console.log('\n=== START BOT ===')

  inSMP = false
  hubReady = false

  bot = createClient({
    host: SERVER.host,
    port: SERVER.port,
    username: SERVER.username,
    offline: true
  })


  bot.on('spawn', () => {

    debug('spawn packet')

    resetDayTimer()

    debug('waiting hub...')

    startHubCheck()

    runSelectorFlow()

  })



  bot.on('text', packet => {

    const msg = packet.message

    if(
      msg.includes('Next shard') ||
      msg.includes('+1 SHARDS')
    ){

      debug('shard gui detected')

      inSMP = true

      if(lastShardLine){
        process.stdout.write('\x1B[2K\r')
      }

      process.stdout.write(msg + '\r')

      lastShardLine = true

    }else{

      console.log(msg)
      lastShardLine = false

    }

  })



  bot.on('disconnect', reason => {

    console.log('\nDISCONNECTED:', reason)

    restartBot()

  })



  bot.on('error', err => {

    console.log('\nERROR:', err.message)

    restartBot()

  })

}



/* ===== SELECT FLOW ===== */

function runSelectorFlow(){

  inSMP = false

  debug('start selector flow')

  // slot 5
  setTimeout(()=>{

    debug('select slot 5')

    hubReady = true

    bot.queue('mob_equipment',{
      hotbar_slot: 4
    })

    bot.queue('inventory_transaction',{
      transaction:{
        legacy:{ legacy_request_id:0 },
        actions:[]
      }
    })

  },8000)



  // move npc
  setTimeout(()=>{

    debug('moving to npc 0.5s')

    bot.queue('player_auth_input',{
      input_data:{ forward:true }
    })

    setTimeout(()=>{

      debug('stop moving')

      bot.queue('player_auth_input',{
        input_data:{ forward:false }
      })

    },500)

  },12000)



  // change slot
  setTimeout(()=>{

    debug('change slot empty')

    bot.queue('mob_equipment',{
      hotbar_slot: 0
    })

  },13500)



  // click npc
  setTimeout(()=>{

    debug('click npc')

    bot.queue('interact',{
      action_id:'mouse_over_entity'
    })

    debug('waiting SMP confirm...')

    startSMPCheck()

  },15000)

}



/* ===== SMP CHECK ===== */

function startSMPCheck(){

  clearTimeout(smpCheckTimer)

  smpCheckTimer = setTimeout(()=>{

    if(!inSMP){

      console.log('\n⚠️ không vào SMP -> retry selector\n')

      debug('retry selector flow')

      runSelectorFlow()

    }else{

      console.log('\n✓ đã vào SMP')

    }

  },20000)

}



/* ===== HUB CHECK ===== */

function startHubCheck(){

  clearTimeout(hubCheckTimer)

  hubCheckTimer = setTimeout(()=>{

    if(!hubReady){

      console.log('\n⚠️ không vào hub')
      console.log('chờ 10 phút...\n')

      debug('hub fail wait 10m')

      setTimeout(()=>{

        restartBot()

      },10*60*1000)

    }

  },15000)

}



/* ===== 24H RESET ===== */

function resetDayTimer(){

  clearTimeout(dayTimer)

  dayTimer = setTimeout(()=>{

    console.log('\n24h restart')

    debug('24h timer reached')

    restartBot()

  },24*60*60*1000)

}



/* ===== RECONNECT ===== */

function restartBot(){

  debug('restart bot')

  clearTimeout(dayTimer)
  clearTimeout(reconnectTimer)
  clearTimeout(smpCheckTimer)
  clearTimeout(hubCheckTimer)

  try{
    bot.close()
  }catch{}

  console.log('reconnect sau 5s...\n')

  reconnectTimer = setTimeout(()=>{

    startBot()

  },5000)

}



/* ===== START ===== */

startBot()
