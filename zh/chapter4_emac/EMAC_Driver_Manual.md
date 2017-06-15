һ��	RT-Thread����̫��EMAC��������
1.	��ʼ������

                                                                     
    eth_system_device_init()
    rt_hw_emac_eth_init();
    lwip_sys_init();
a.	eth_system_device_init����������Rx��Tx�����߳�,ͬʱ���������������������ݰ���ͬ�����պͷ��ͣ��ú����ﱻ�����̵߳�stack��С���߳����ȼ�ͳһ��rtconfig.h��������á�
b.	rt_hw_emac_eth_init����Ҫ�ص���ܵĺ���,��ʵ�������е�EMAC��phy��ʼ������������ͨ������Ĵ�����������γ�ʼ��emac��phy��
int rt_hw_ emac _eth_init(void)
{
    struct rt_ emac _eth * emac _eth_device;
   
    emac_eth0_ptr->EMACx=(Emac_EMAC_REG_GRP*)Emac_EMAC0_REG_GRP_PTR;
    emac _eth0_ptr->plugged    = 0;
    emac _eth0_ptr->tx_vector  = EMC0_TX_IRQn;
    emac _eth0_ptr->rx_vector  = EMC0_RX_IRQn;
    init_emac0_rx_ring_buffer();
    emac _eth_device = & emac _eth0_device;
    emac _eth_device->ETHx = emac_eth0_ptr;
    rt_memcpy(emac_eth_device->dev_addr,emac_eth0_addr,sizeof(emac_eth0_addr));

    /* register interrupt */
	  rt_hw_interrupt_install(emac_eth_device->ETHx->tx_vector,ETH_TX_IRQHandler,emac_eth_device,"e0txisr");
     rt_hw_interrupt_set_priority(emac_eth_device->ETHx->tx_vector,IRQ_LEVEL_1);
	  rt_hw_interrupt_install(emac_eth_device->ETHx->rx_vector,ETH_RX_IRQHandler,emac_eth_device,"e0rxisr");
    rt_hw_interrupt_set_priority(emac_eth_device->ETHx->rx_vector,IRQ_LEVEL_1);
    /* RT ETH device interface init */
    emac _eth_device->parent.parent.init       = rt_ emac _eth_init;
    emac _eth_device->parent.parent.open       = rt_ emac _eth_open;
    emac _eth_device->parent.parent.close      = rt_ emac _eth_close;
    emac _eth_device->parent.parent.read       = rt_ emac _eth_read;
    emac _eth_device->parent.parent.write      = rt_ emac _eth_write;
    emac _eth_device->parent.parent.control    = rt_ emac _eth_control;
    emac _eth_device->parent.parent.user_data  = RT_NULL;
    emac _eth_device->parent.eth_rx            = rt_ emac _eth_rx;
    emac _eth_device->parent.eth_tx            = rt_ emac _eth_tx;

    /* Init tx buffer free semaphore */
    rt_sem_init(&emac _eth_device->tx_buf_free, "eth0_tx", TX_DESCRIPTOR_NUM, RT_IPC_FLAG_FIFO);

    /* Register eth device */
    eth_device_init(&(emac _eth_device->parent), "e0");
    eth_init(emac _eth_device->ETHx, emac _eth_device->dev_addr);
   
    /* Creat a timer for monitor the link status */
	  rt_timer_init(&(emac _eth_device->link_timer), "link_timer", 
		eth_chk_link, 
		(void *) emac _eth_device, 
		RT_TICK_PER_SECOND, 
		RT_TIMER_FLAG_PERIODIC);

	  rt_timer_start(&(emac _eth_device->link_timer));
    rt_hw_interrupt_umask(emac _eth_device->ETHx->tx_vector);
    rt_hw_interrupt_umask(emac _eth_device->ETHx->rx_vector);
    rt_kprintf("ETH0 init Completed ....\n");
    return 0;
}

�����ǿ�ʼ�˽�ú���֮ǰ���������˽��¼�����Ҫ�����ݽṹ��
a.	emac _eth0_device
emac _eth0_device��ԭ��Ϊ:
struct rt_ emac _eth
{
    /* inherit from ethernet device */
    struct eth_device parent;
    ETH_STR *ETHx;
    uint8_t dev_addr[6];
    struct rt_semaphore tx_buf_free;
	  struct rt_timer     link_timer;
};
�ýṹ�����parent����ԭ����ethernetif.h��,�û��ڷ�װ�Լ�����������ʱ���붨��ò���,����ע�������豸ʱ��Ҫ�õ���
ETH_STRΪ��EMAC��ص����Զ���:
typedef struct _eth_str
{
    EMAC_REG_GRP *EMACx;

    struct eth_descriptor rx_desc[RX_DESCRIPTOR_NUM];
    struct eth_descriptor tx_desc[TX_DESCRIPTOR_NUM];
    volatile struct eth_descriptor  *cur_tx_desc_ptr, *cur_rx_desc_ptr, *fin_tx_desc_ptr;
    rt_uint8_t rx_buf[RX_DESCRIPTOR_NUM][PACKET_BUFFER_SIZE];
    rt_uint8_t tx_buf[TX_DESCRIPTOR_NUM][PACKET_BUFFER_SIZE];
    rt_int32_t plugged;
    rt_int32_t tx_vector,rx_vector;
}ETH_STR;
Emac_EMAC_REG_GRP *EMACxΪEMAC�Ĵ�����ָ��,��������Rx��Tx����������,��������״̬����,Tx��Rx�ж������Ŷ��塣�ýṹ������ݿɸ����û���EMACʵ����������壬û���ϸ��Ҫ������ơ��е�EMACҪ��������Ӧ�ö����ڷ�Cache�ڴ����䡣
dev_addr���MAC��ַ��
tx_buf_free��link_timerΪ�����Զ���ı���������Ҫ���û����붨�塣
���,EMAC�����ݽṹ�ķ�װ����struct eth_device parent�Ǳ��붨����⣬�����ı����ɸ����û���ʵ����������з�װ��
rt_hw_ emac _eth_init������ʼ����EMAC�ļĴ�����,��̫��������״̬��Tx�жϺ�,Rx�жϺ�,Ȼ���ʼ��Tx��Rx�жϲ������жϳ�ʼ����������:
/* register interrupt */
rt_hw_interrupt_install(emac_eth_device->ETHx->tx_vector,ETH_TX_IRQHandler, emac _eth_device,"e0txisr");
    rt_hw_interrupt_set_priority(emac_eth_device->ETHx->tx_vector,IRQ_LEVEL_1);    rt_hw_interrupt_install(emac_eth_device->ETHx->rx_vector,ETH_RX_IRQHandler, emac _eth_device,"e0rxisr");
    rt_hw_interrupt_set_priority(emac _eth_device->ETHx->rx_vector,IRQ_LEVEL_1);
    emac _eth_device->ETHx->tx_vectorΪTx�ж������š�
    ETH_TX_IRQHandlerΪTx�жϷ������
    emac_eth_deviceΪEMAC��װ�õĽṹ��,��Ϊ���������ETH_TX_IRQHandlerʹ�á�
    "e0txisr"Ϊ�жϷ����������ơ�
    ���������жϲ�����,����������ʹ���ж�,��Ϊ��ص�EMAC����������RT-Thread����̫���ܹ���û�г�ʼ���á�
    �ڳ�ʼ��EMAC���жϲ�����,������������EMAC�ṹ����parent�Ĳ���,������´��롣
    /* RT ETH device interface init */
    emac _eth_device->parent.parent.init       = rt_emac_eth_init;
    emac _eth_device->parent.parent.open       = rt_emac_eth_open;
    emac _eth_device->parent.parent.close      = rt_emac_eth_close;
    emac _eth_device->parent.parent.read       = rt_emac_eth_read;
    emac _eth_device->parent.parent.write      = rt_emac_eth_write;
    emac _eth_device->parent.parent.control    = rt_emac_eth_control;
    emac _eth_device->parent.parent.user_data  = RT_NULL;

    emac _eth_device->parent.eth_rx            = rt_emac_eth_rx;
    emac _eth_device->parent.eth_tx            = rt_emac_eth_tx;
���Ƿ������ϴ���������Linux��file_operations��ʼ����
rt_emac_eth_init������ɽ���EMAC��PHY�ĳ�ʼ���������ڴˡ�
rt_emac_eth_open��rt_emac_eth_close��rt_emac_eth_read��rt_emac_eth_write��rt_emac_eth_control�ɰ�����������Ĵ����ʽ��д��
static rt_err_t rt_stm32_eth_open(rt_device_t dev, rt_uint16_t oflag)
{
return RT_EOK;
}
static rt_err_t rt_stm32_eth_close(rt_device_t dev)
{
	return RT_EOK;
}

static rt_size_t rt_stm32_eth_read(rt_device_t dev, rt_off_t pos, void* buffer, rt_size_t size)
{
	rt_set_errno(-RT_ENOSYS);
	return 0;
}

static rt_size_t rt_stm32_eth_write (rt_device_t dev, rt_off_t pos, const void* buffer, rt_size_t size)
{
	rt_set_errno(-RT_ENOSYS);
	return 0;
}

static rt_err_t rt_stm32_eth_control(rt_device_t dev, rt_uint8_t cmd, void *args)
{
	switch(cmd)
	{
	case NIOCTL_GADDR:
		/* get mac address */
		if(args) rt_memcpy(args, stm32_eth_device.dev_addr, 6);
		else return -RT_ERROR;
		break;

	default :
		break;
	}

	return RT_EOK;
}
rt_emac_eth_rx�Ǵ����������ȡ���ݴ洢��lwIP�Ļ�������


/* reception packet. */
struct pbuf *rt_emac_eth_rx(rt_device_t dev)
{
    struct pbuf* p;
    rt_uint32_t framelength = 0;
    register rt_base_t level = 0;
    unsigned char *buffer_addr = 0;

    /* init p pointer */
    p = RT_NULL;

    /* disable interrupt */
    level = rt_hw_interrupt_disable();
    
    if (emac0_ring_buffer.rx_cnt > 0)
    {
        emac0_ring_buffer.rx_cnt--;
        buffer_addr = emac0_ring_buffer.buffer_rx_out_ptr->buffer;
        framelength = emac0_ring_buffer.buffer_rx_out_ptr->packet_length;
        emac0_ring_buffer.buffer_rx_out_ptr++;
        
        if(emac0_ring_buffer.buffer_rx_out_ptr == &emac0_ring_buffer.rx_buff[RX_RING_BUFFER_LENGTH])
        {
            emac0_ring_buffer.buffer_rx_out_ptr   = &emac0_ring_buffer.rx_buff[0];
        }
    }else{
        rt_hw_interrupt_enable(level);
        return p;
    }

    /* allocate buffer */
    p = pbuf_alloc(PBUF_RAW, framelength, PBUF_POOL);
    if (p != RT_NULL)
    {
        rt_uint8_t * from;

        from = (unsigned char *)(buffer_addr);

        pbuf_take(p, from, framelength);
     
#ifdef ETH_RX_DUMP
        packet_dump("RX dump", p);
#endif /* ETH_RX_DUMP */
    }
    rt_hw_interrupt_enable(level);
    return p;
}
�����EMAC��DMA�������������ݽṹ�Լ���ȡ���ݵķ�ʽ��һ��,���ﲻ����ζ�ȡ������������,�ص������ν����ݴ洢��lwIP�Ļ�������
struct pbuf* p;

/* allocate buffer */
    p = pbuf_alloc(PBUF_RAW, framelength, PBUF_POOL);
    if (p != RT_NULL)
    {
        rt_uint8_t * from;

        from = (unsigned char *)(buffer_addr);

        pbuf_take(p, from, framelength);
     
#ifdef ETH_RX_DUMP
        packet_dump("RX dump", p);
#endif /* ETH_RX_DUMP */
    }
    rt_hw_interrupt_enable(level);
return p;

pbuf_alloc(PBUF_RAW, framelength, PBUF_POOL);��lwIP�Ļ������з���framelength��С���ڴ�,framelengthΪʵ�ʽ��յ������ݰ���С,��ע��PBUF_RAW��PBUF_POOL�����ͣ����ò������п�����ɽ��պͷ����쳣,����ʹ�õ���lwIP��memory pool�ڴ���䷽ʽ������ʹ��������������
pbuf_take�����յ������ݰ�copy��lwIP�Ļ�����,��lwIP��IP��ȥ���������
rt_emac_eth_tx������lwIP IP������ݰ��洢������������,��EMAC��DMA��ȡ���ͳ�ȥ��
/* ethernet device interface */
/* transmit packet. */
rt_err_t rt_emac_eth_tx( rt_device_t dev, struct pbuf* p)
{
    register rt_base_t level = 0;
    rt_uint16_t length = 0;

     /* Disable interrupt */
     level = rt_hw_interrupt_disable();
     if (emac0_ring_buffer.tx_cnt <TX_RING_BUFFER_LENGTH)
     {
        emac0_ring_buffer.tx_cnt++;
        length = p->tot_len;
        if (length > 1518)
        length = 1518;
        
        pbuf_copy_partial(p, ( void *)emac0_ring_buffer.buffer_tx_in_ptr->buffer, length, 0);
        emac0_ring_buffer.buffer_tx_in_ptr->packet_length = length;
        emac0_ring_buffer.buffer_tx_in_ptr++;

        if(emac0_ring_buffer.buffer_tx_in_ptr== &emac0_ring_buffer.tx_buff[TX_RING_BUFFER_LENGTH])
        {
            emac0_ring_buffer.buffer_tx_in_ptr   = &emac0_ring_buffer.tx_buff[0];
        }
        
        rt_hw_interrupt_enable(level);
        rt_sem_release(&emac0_ring_buffer.tx_buf_available_sem);

        /* Return SUCCESS */
        return RT_EOK;
    }

    rt_hw_interrupt_enable(level);
    return -RT_ERROR;
}
�ú�����ȡlwIPЭ��IP��ָ�����ȵ����ݰ��洢��EMAC�ķ������������乩DMA��ȡ��������̫���硣pbuf_copy_partial(p, ( void *)emac0_ring_buffer.buffer_tx_in_ptr->buffer, length, 0)��p��Ϊ�βΰ���Ҫ�������ݰ��ĳ���;( void *)emac0_ring_buffer.buffer_tx_in_ptr->bufferΪ����������������;lengthΪҪ�������ݰ��ĳ���,��p�ж�ȡ;���һ������Ϊoffset��ƫ��Ϊ0������ͷ��ʼȡ���ݡ���ͬEMAC��DMA���ݽṹ���������Ķ��巽ʽ��һ��,���ݷ�������������DMA�Ĳ�����һ��Ҳ�Ͳ�һ����,���ﲻ��������һ���Ĳ�����
rt_sem_init��ʼ���ź�������Tx�жϷ��������rt_emac_eth_tx֮���ͬ����
eth_device_initע�������豸,e0Ϊ�豸��,��ע������ǿ����parent�������õ�, eth_device_init����λ��ethernetif.c�ļ��С�
eth_init������ʼ��EMAC��PHY,�����ᵽ���ɽ��ú���ʵ�ֵ�ȫ����������rt_emac_eth_init�����С�
rt_timer_init������һ����ʱ���������ڼ���������·״̬,������߻����½�ͨ���硣�������ǽ���Ҫ�����ú��������á�

 /* Creat a timer for monitor the link status */
	rt_timer_init(&(emac_eth_device->link_timer), "link_timer", 
		eth_chk_link, 
		(void *)emac_eth_device, 
		RT_TICK_PER_SECOND, 
		RT_TIMER_FLAG_PERIODIC);

	rt_timer_start(&(emac_eth_device->link_timer));
�����ص㿴eth_chk_link����

void eth_chk_link(void *param)
{
    uint8_t  phy_addr = CONFIG_ETH0_PHY_ADDR;
    uint32_t reg      = 0;
    struct rt_emac_eth *emac_eth_device = (struct rt_emac_eth *)0 ;
    ETH_STR *ETHx = (ETH_STR *)0;

    emac_eth_device = (struct rt_emac_eth *)param;
    ETHx = (ETH_STR *)(emac_eth_device->ETHx);
    

    if(ETHx->EMACx == Emac_EMAC0_REG_GRP_PTR)
    {
        phy_addr = CONFIG_ETH0_PHY_ADDR;
    }else{
        phy_addr = CONFIG_ETH1_PHY_ADDR;
    }

    reg = mdio_read(ETHx,phy_addr, MII_BMSR);

    if (reg & BMSR_LSTATUS)
    {
        if (!ETHx->plugged)
        {
            ETHx->plugged = TRUE;

            reg = mdio_read(ETHx,phy_addr, MII_LPA);

            if (reg & ADVERTISE_100FULL)
            {
                ETH_DBG("100 Full\n");
                ETHx->EMACx->emac_mcmdr |= MCMDR_100M_FULL_MSK;
            }
            else if (reg & ADVERTISE_100HALF)
            {
                ETH_DBG("100 Half\n");
                ETHx->EMACx->emac_mcmdr &= (~MCMDR_FDUP_MSK);
                ETHx->EMACx->emac_mcmdr |= MCMDR_100M_HALF_MSK;
            }
            else if (reg & ADVERTISE_10FULL)
            {
                ETH_DBG("10 Full\n");
                ETHx->EMACx->emac_mcmdr &= ~MCMDR_OPMOD_MSK;
                ETHx->EMACx->emac_mcmdr |= MCMDR_FDUP_MSK;
            }
            else
            {
                ETH_DBG("10 Half\n");
                ETHx->EMACx->emac_mcmdr &= (~MCMDR_100M_FULL_MSK);
            }
            ETHx->EMACx->emac_mcmdr  |= (MCMDR_RXON_MSK|MCMDR_TXON_MSK);    

            /* Maybe some income packets in buffer */
            ETH_TRIGGER_TX(ETHx->EMACx);

            /* Send link up. */
            eth_device_linkchange(&(emac_eth_device->parent), RT_TRUE);
        }
    }
    else
    {
        if (ETHx->plugged)
        {
            ETH_DBG("Link Down\n");
            ETHx->plugged = FALSE;
            rt_hw_interrupt_mask(ETHx->tx_vector);
            rt_hw_interrupt_mask(ETHx->rx_vector);

            /* Disable Tx and Rx */
            ETHx->EMACx->emac_mcmdr  &= (~(MCMDR_RXON_MSK|MCMDR_TXON_MSK));    

            /* Send link down. */
            eth_device_linkchange(&(emac_eth_device->parent), RT_FALSE);
        }
    }
}
�������ϴ������ǿ��Կ�������Χ��PHY��״̬�Ĵ������ж�����Ĺ���״̬,������״̬�Ƿ�仯�������eth_device_linkchange����,��2������RT_TRUE��ʾ��·����,RT_FALSE��ʾ��·���������ú����������û�������Ϊһ���߳�,���ȼ�������ñȽϵ�,����ʱ�������ñȽϳ��Է�ֹ��·����ʱ���������Ƶ�������л�������ϵͳ���ܲ���Ӱ�졣
�ڳ�ʼ�����,�������ڿ���ʹ���ж��ˡ�
    rt_hw_interrupt_umask(emac_eth_device->ETHx->tx_vector);
    rt_hw_interrupt_umask(emac_eth_device->ETHx->rx_vector);
    RT-ThreadΪϵͳ��������������ṩ��һ�ֳ�ʼ������,�û�ֻ��Ҫ�ں�����}ĩβ����INIT_COMPONENT_EXPORT(������);RT-Thread�����Զ�������Щ����ִ��,Ϊ�˷������������������������������ĩβȥ�������,���û��ֶ�����������������
    eth_system_device_init();
    rt_hw_nuc970_eth_init();
    lwip_sys_init();
     �������ݽ����������RT-Thread����̫�������ʵ��EMAC�ĳ�ʼ��,������û�н�����γ�ʼ��PHY��EMAC,Ҳû����ϸ������γ�ʼ����������DMA,��ͬ��EMAC��PHY���ǵĳ�ʼ����ʽ�����̲�һ����
2.	EMAC ISR How To?
    EMAC���յ�һ��packet������һ��Rx �ж�,�ڷ�����һ��packet������һ��Tx�жϡ��е�MCU or MPU��������IRQ�ֳ������ж�����������,�е�MCU or MPU��һ���ж�����������;��������һ��IRQ��������IRQ�������Ƕ��ֳ���������������RT-Thread����µ�EMAC ISRӦ������Щ������
a.	Rx ISR
    Rx ISR����EMAC�ļܹ���������жϱ�־λ����ص�״̬,����Ҫע����մ����жϱ�־λ�Ĵ���,������ɵ����ϵ��鷳��Ȼ�����EMAC��DMA�������������ݴ��������ڽ��յ�һ����Ч�İ���͵���eth_device_ready(&(emac_eth->parent))����,����һ������֪ͨrt_nuc970_eth_rx���������������������������
b.	Tx ISR
    rt_nuc970_eth_tx������ȡ��lwIP IP������ݰ���洢��������������,Ȼ������EMAC�ķ��͹���,EMAC������һ�����ݰ�������һ���жϡ���Tx ISR��Ӧ�������EMAC״̬�ͷ����жϱ�־λ�Ĺ�������EMAC�����ѯ�����������Ƿ�������δ����,Tx ISR����ѯ�����������Ƿ���δ���͵�����,���䷢�ͳ�ȥ��
