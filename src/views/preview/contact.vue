// pages/my/about/about.js
Page({
  /**
   * 页面的初始数据
   */
  data: {
    array: ["在线客服", "电话客服"],
  },

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {},
  /* 点击协议 */
  clickAgreement() {
    wx.navigateTo({
      url: "/pages/my/agreement/agreement",
    });
  },
  /* 选择打电话 */
  bindTel(e) {
    var now = new Date();
    var dayOfWeek = now.getDay(); // 0（星期日）到 6（星期六）
    var hour = now.getHours();
    console.log(hour);
    var isWorkingDay = dayOfWeek > 0 && dayOfWeek < 6; // 周一到周六
    var isWorkingHour = hour >= 9 && hour < 18; // 上午9点到下午18点
    if ((isWorkingDay && isWorkingHour) || (dayOfWeek === 6 && isWorkingHour)) {
      wx.makePhoneCall({
        phoneNumber: "18302821638", // 替换为电话号码
      });
    } else {
      wx.showToast({
        title: "请工作日9:00-18:00拨打",
        icon: "none",
        duration: 3000,
      });
    }
  },
  /**
   * 生命周期函数--监听页面初次渲染完成
   */
  onReady: function () {},

  /**
   * 生命周期函数--监听页面显示
   */
  onShow: function () {},

  /**
   * 生命周期函数--监听页面隐藏
   */
  onHide: function () {},

  /**
   * 生命周期函数--监听页面卸载
   */
  onUnload: function () {},

  /**
   * 页面相关事件处理函数--监听用户下拉动作
   */
  onPullDownRefresh: function () {},

  /**
   * 页面上拉触底事件的处理函数
   */
  onReachBottom: function () {},

  /**
   * 用户点击右上角分享
   */
  onShareAppMessage: function () {},
});
